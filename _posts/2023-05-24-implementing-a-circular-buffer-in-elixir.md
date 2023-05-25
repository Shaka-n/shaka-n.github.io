---
layout: post
title: Implementing a Circular Buffer in Elixir
categories: Elixir
---

I recently completed an [Exercism challenge](https://exercism.org/tracks/elixir/exercises/circular-buffer) to implement a [circular buffer](https://en.wikipedia.org/wiki/Circular_buffer) in Elixir, and I wanted to talk through my solution. I've pasted the written instructions for the buffers behavior below, though the test suite had some additional complexity which I'll try to explain.

---

**Instructions**

A circular buffer, cyclic buffer or ring buffer is a data structure that uses a single, fixed-size buffer as if it were connected end-to-end.

A circular buffer first starts empty and of some predefined length. For example, this is a 7-element buffer:

[ ][ ][ ][ ][ ][ ][ ]
Assume that a 1 is written into the middle of the buffer (exact starting location does not matter in a circular buffer):

[ ][ ][ ][1][ ][ ][ ]
Then assume that two more elements are added — 2 & 3 — which get appended after the 1:

[ ][ ][ ][1][2][3][ ]
If two elements are then removed from the buffer, the oldest values inside the buffer are removed. The two elements removed, in this case, are 1 & 2, leaving the buffer with just a 3:

[ ][ ][ ][ ][ ][3][ ]
If the buffer has 7 elements then it is completely full:

[5][6][7][8][9][3][4]
When the buffer is full an error will be raised, alerting the client that further writes are blocked until a slot becomes free.

When the buffer is full, the client can opt to overwrite the oldest data with a forced write. In this case, two more elements — A & B — are added and they overwrite the 3 & 4:

[5][6][7][8][9][A][B]
3 & 4 have been replaced by A & B making 5 now the oldest data in the buffer. Finally, if two elements are removed then what would be returned is 5 & 6 yielding the buffer:

[ ][ ][7][8][9][A][B]
Because there is space available, if the client again uses overwrite to store C & D then the space where 5 & 6 were stored previously will be used not the location of 7 & 8. 7 is still the oldest element and the buffer is once again full.

[C][D][7][8][9][A][B]

---


The challenge itself encourages you to leverage Elixir GenServers in your solution, but Agents or even vanilla Processes would be equally valid.  I `use GenServer` at the top of my module to inject the GenServer quality of life functions, and then I began implementing the callbacks necessary for using them in the provided functions that the test suite will be testing against. On my first pass, my instinct was to leverage pattern matching on lists to manipulate the ordering of the list. I liked the elegance of this initial solution, but after sleeping on it I realized it could be much more performant. The next day I implemented a second solution, one which is less elegant but certainly faster and more scalable.

As I said, my gut instinct was to leverage pattern matching. Lists in Elixir are effectively linked lists, meaning that internally they are pairs containing the head and the tail of the list. Even a list of one item is actually a list where the head is that item and the tail is an empty list. If I can ensure that the head of the list is always the oldest element, I can enforce a predictable order on the list. In order to do this, I need to implement an insertion method that always adds new items at the very end of the tail. I achieved this through recursively calling `insert/2`, pattern matching on the head and tail until the base case is reached (i.e. one item and an empty list). By inserting this way, the oldest item will always be the head of the list. This makes reading from the list simple, as we simply pattern match on the list, returning the head as the value and the tail as the new state. We can enforce our fixed size by returning an error if the length of our list reaches our capacity. 

After this all we have left to do is implement the overwrite behavior. We achieve this be simply structuring the callback in a slightly different way to the basic write callback. If we've reached capacity, we pattern match on the tail and insert the item using that tail, throwing out the old head. If it hasn't reached capacity, we can insert as we would normally.

```
defmodule CircularBuffer do
  @moduledoc """
  An API to a stateful process that fills and empties a circular buffer
  """
  use GenServer
  @impl true
  def init(capacity) do
    {:ok, {capacity, []}}
  end
  @impl true
  def handle_call(:read, _from, state) do
    case state do
      {_capacity, []} ->
        {:reply, {:error, :empty}, []}
      {capacity, [oldest | tail]} ->
        {:reply, {:ok, oldest}, {capacity, tail}}
    end
  end
  @impl true
  def handle_call({:write, item}, _from, {capacity, list} = _state) do
    if length(list) == capacity do
        {:reply, {:error, :full}, list}
    else
      {:reply, :ok, {capacity, insert(list, item)}}
    end
  end
  @impl true
  def handle_call({:overwrite, item},
                 _from, {capacity, 
                [ _head | tail] = list} = _state) 
                do
    if length(list) == capacity do
      {:reply, :ok, {capacity, insert(tail, item)}}
    else
      {:reply, :ok, {capacity, insert(list, item)}}
    end
  end
  defp insert([], item), do: [item]
  defp insert([head | []], item), do: [head| [item | []]]
  defp insert( [head | tail] = _list, item), do: [head | insert(tail, item)]

  @impl true
  def handle_cast(:clear, {capacity, _list} = _state) do
    {:noreply, {capacity, []}}
    end
end
```

This solution has the benefit of being pretty to look at and taking up relatively few lines of code. It also has a blazing fast read time, since the next item we want is always at the front. However, there are a couple of significant drawbacks. For starters, my insertion strategy scales very poorly. Every time I want to insert something I have traverse the entire buffer until I get to the end. For a small buffer this is trivial, but if the buffer grew to thousands or tens of thousands of items in length, I would be waiting around a lot to insert new items. On top of that, by leaning on `length()` to determine if I'm at capacity, I'm potentially adding another bottleneck. Under the hood this is an Erlang function and is probably quite fast, but at time of writing I'm not familiar with the Erlang libraries to be certain. A worst case scenario though might have me traversing the entire list twice in a single `write` call. While this is still only 2N in Big O, we can definitely do better.


My second solution is fairly simple in concept, relying on the speed of key lookup in maps, but is a little less easy on the eyes:
```
defmodule CircularBuffer do
  @moduledoc """
  An API to a stateful process that fills and empties a circular buffer
  """
  use GenServer
  @doc """
  Initializes buffer with max capacity, a read index,
  a write index, an available slot counter, and a data store.
  """
  @impl true
  def init(capacity) do
    {:ok, %{cap: capacity, 
            next_read: 1, 
            next_write: 1, 
            slots: capacity, 
            store: init_store(capacity)}
    } 
  end

  defp init_store(capacity) do
    Enum.reduce(1..capacity, %{}, fn x, acc -> Map.put(acc, x, nil) end)
  end

  @impl true
  def handle_call(:read, _from, %{next_read: next_read, store: store} = state) do
    case store[next_read] do
      nil ->
        {:reply, {:error, :empty}, state}
      value ->
        {:reply, {:ok, value}, update_read(state)}
    end
  end

  @impl true
  def handle_call({:write, item}, _from, %{slots: slots} = state) do
    if slots == 0 do
      {:reply, {:error, :full}, state}
    else
      {:reply, :ok, update_write(state, item)}
    end
  end

  @impl true
  def handle_call({:overwrite, item}, _from, %{slots: slots} = state) do
    if slots == 0 do
      {:reply, :ok, update_overwrite(state, item)}
    else
      {:reply, :ok, update_write(state, item)}
    end
  end

  @impl true
  def handle_cast(:clear, state) do 
    {:noreply, %{state | next_read: 1, 
                        next_write: 1, 
                        slots: state.cap, 
                        store: init_store(state.cap)
                }
    }
    end

  defp update_read(%{cap: cap,
    next_read: current_next_read,
    slots: slots,
    store: store} = state) 
  do
      if current_next_read == cap do
      %{state | next_read: 1, 
                slots: slots + 1, 
                store: %{store | current_next_read => nil}
        }
    else
      %{state | next_read: current_next_read + 1, 
                slots: + 1, 
                store: %{store | current_next_read => nil}
        }
    end
  end

  defp update_write(%{cap: cap, 
                    next_write: next_write, 
                    slots: slots, 
                    store: store} = state, item) do
    if next_write == cap do
      %{state | next_write: 1, 
                slots: slots - 1, 
                store: %{ store | next_write => item}
        }
    else
      %{state | next_write: next_write + 1, 
                slots: slots - 1, 
                store: %{store | next_write => item}
        }
    end
  end

  defp update_overwrite(%{cap: cap, 
                        next_write: next_write, 
                        store: store, 
                        next_read: next_read}= state, item) do
    incoming_next_read = 
        if next_read + 1 > cap do 
            1
        else 
            next_read + 1
        end
    if next_write == cap do
      %{state | next_read: incoming_next_read, 
                next_write: 1, 
                store: %{ store | next_write => item}}
    else
      %{state | next_read: incoming_next_read, 
                next_write: next_write + 1, 
                store: %{store | next_write => item}}
    end
  end
end
```
This solution relies on using lookup keys to keep track of where the next item goes, where the next item to read goes, and how many available slots we have. The crux of the solution lies in the init function. I use a range of `1..capacity` to create a map where the keys are integers pointing to null values. Our `next_read` and `next_write` keys keep track of where in the buffer we need to read and write next. This solution is much, much faster, with a worst case Big O of N, where we have to create our buffer of N size once, but never have to fully traverse it again. As we insert and extract items, we update the pointers. 

A further improvement on this solution might be to use Streams to lazily load in new chunks of our buffer, rather than build it all at once. Or, more simply, we could simply make the extension of our store be a function of the insertion. Done this way, the store map would begin empty and only have new keys added when a write is performed, emitting an error when a normal write is attempted when our available slots is 0. I believe we could achieve a O(1) with this solution, which is pretty cool.

This was a fun one, and an interesting challenge that still has me thinking of ways to improve on my solutions. Check out a slightly more readable version on my [GitHub](https://github.com/Shaka-n/elixir-exercism-exercises/blob/main/circular-buffer/lib/circular_buffer.ex) or [Exercism](https://exercism.org/tracks/elixir/exercises/circular-buffer/solutions/Shaka-n) profiles.

