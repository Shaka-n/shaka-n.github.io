---
title: Manipulating Bitstrings in Elixir
category: Elixir, Quick Tips
--- 

Per the [official documentation](https://elixir-lang.org/getting-started/binaries-strings-and-char-lists.html#bitstrings), bitstrings are a fundamental data type in Elixir representing a contiguous sequence of bits in memory. Useful for UTF-8 encodings and other fun things like secret obfuscation, bitstrings are a great tool to have in your back pocket. Instead of parroting existing documentation, I wanted to share a little nuance that I had trouble grasping while I was learning about this data type.

When you concatenate, prepend, or append bitstring literals in a bitstring special form(i.e. <<value::size>>), the output format can look very different from the input format. If your output produces a valid UTF-8 encoded character, you will get that character. If the result of the concatentation would overflow 8 bits (the maximum), then the result will be whatever integer is represented by that 8-bit value, and then a bitstring literal comprised of the remainder. For example:

```
value = <<0b110::3, 0b001::3>>
new_value = <<0b011::3, value::bitstring, 0b000::3>>
# => <<120, 8::size(4)>>
```

In this example we are both prepending and appending bitstring literals to the original bitstring value, which you can visualize like so:

```
<<0b011::3, 0b110::3, 0b001::3, 0b000::3>>
```

You'll notice that there are more than 8 bits in this intermediate representation. Since we obviously cannot have more than 8 bits in a bitstring, we need to do something with the extra bits. Elixir will take the first 8 bits provided and return them as an integer(i.e. 120). The remaining bits will be returned as a bitstring literal, using the verbose syntax (i.e. 8::size(4)). Taking it further, what happens if we have many more bits? 

```
<<0b011::3, 0b110::3, 0b001::3, 0b000::3, 0b001::3, 0b010::3>>
# => <<120, 130, 2::size(2)>>
```

So you see, bitstrings concatenated/appended/prepended in this way will return integers from the leading 8-bit fragments, and return the remaining bits as bitstring literals in the verbose syntax.

This was very confusing for me when I first learned it, so I hope this helps someone. This is by no means a complete explanation of bitstrings, so for more on bitstrings, check out the [official docs](https://elixir-lang.org/getting-started/binaries-strings-and-char-lists.html).