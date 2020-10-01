---
layout: post
title:  "Ruby Enumerables Series: Chain"
date:   2020-09-27 16:11:00 -0400
categories: Ruby
---
Ruby Enumerables Series: Chain

This week I'm continuing the enumerables series with chain. It's a relatively simple enumerable, so this may be an entry that I return to to flesh out later. I have a hunch that there are a number of methods that are logically connected with each other that aren't necessarily next to each other in the docs. So if for some reason somebody is keeping track, I might return to this one to flesh it out more with references to related methods.

Moving on, chain is slightly meta, in that it allows you to return an enumerato object from an enumerator and an enumerable. In Ruby, the Enumerator class which allows for internal or external iteration. The Iterator is a design pattern that goes beyond the scope of this blog post, but suffice to say that an external iterator is something that acts on a collection from outside, and an internal iterator is something that acts on a collection from within the collection. Said differently, an external iterator is a method waiting for a collection to be passed to it as an argument, while an internal iterator is a method that is already available to a collection that allows it to traverse through itself. I would love to go on, but I have to find a way to fit some code in here.

Basically, you invoke chain on an enumerable and you give chain another enumerable as an argument. It will return an instance of the Enumerator class which can be converted to an array or hash depending on your needs. It chains the two enumerables together.

Most simply, this can be used to combine two arrays:

```
array1 = [1,2,3]
array2 = [4,5,6]

enumerator = array1.chain(array2)

enumerator.to_a

# => [1, 2, 3, 4, 5]

```

You can also use it to chain an array and a hash (they are both enumerables), but the results are a little odd:
```

array = [1,2,3]
hash = {a => 4, b => 5}

enumerator = array.chain(hash)

enumerator.to_a

# => [1, 2, 3, ['a', '4'], ['b', '5']]
```

Attempting to convert the enumerator to a hash throws an error, so you can only convert this to an array. For now I can only speculate as to when I might want to add hash values to an array, but preserve the relation between key and value by nesting them together in a sub-array. It would be less performant in any case that I can think of, so if anyone can think of a time that they would do this, please drop me a line. Maybe I'm just missing something.

For now, that's chain. Next week I'm thinking of taking a crack at Sass, so hopefully we'll get a longer post out of that.


