---
layout: post
title:  "Ruby Enumerables Series: Any or All?"
date:   2020-09-27 06:35:01 -0400
categories: Ruby
---
Ruby Enumerables Series: Any or All?

Last week I wrote about the reduce method and how it's implemented in Javascript. Continuing on that theme, let's switch gears and look at the baked in enumerables found in Ruby. Ruby's Enumerable mixin has a ton of unique, built-in methods that can make your life a whole lot easier if you take the time to learn when to use them. I barely even know 10% of them, which is why I'm going to try and least learn what they all do at a glance. In all, there are 59 unique instance methods, but some are just inversions of other methods, so I'm not going to write an entire blog for each. Instead, I'm going to do my best to group the most closely related methods together and explain them in assocation with one another. First up: any? and all?.

All is a very simple method: it accepts a code block, passes each element in the collection to the block in turn, and returns true if the block never returns false or nil. This might look like this:

```
array = [1,4,7,9]

array.all? {|n| n < 10}

```

This will return true because all of the elements are less than 10. Easy. It has an even simpler implementation. If you pass an array to all? without a code block, it will return true as long as none of the elements are false or nil.

Inverting this concept, any? accepts a code block, passes each element in the collection to the block in turn, and returns true if the block ever returns false or nil. In the above example, using any? would return false. However, the below example will return true:

```

array = [1,2,3,"4"]

array.any? {|n| n.is_a?(String)}

```

Here all we are checking for is whether or not any of the elements are strings. Since the fourth element is the string "4", any? returns True. Similarly to all?, any? can also be used without providing a code block. In this case, it will return true if at least one of the elements is not false or nil.

I should make a note of the third usage for any? and all? tha has only been added: pattern matching. Pattern matching is a bit beyond the scope of this blog entry, but in essence, it allows you to provide a pattern with certain known values and certain unknown values to be expected by a block of code. I myself don't full grasp the power of this, but my understanding is that it allows you to be much more concise, which of course we love. In the case of all?, it will return whether the expected pattern matches for every collection member. In the case of any? it will return wehter the expected pattern matches for any collection member.

That's any and all. Perhaps next week we'll dive further into pattern matching.


