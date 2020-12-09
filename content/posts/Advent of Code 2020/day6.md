---
title: "Day 6: Customs"
date: 2020-12-06T12:06:25+01:00
hero: /posts/Advent of Code 2020/aoc.jpeg
description: Day 6 of AoC 2020
menu:
  sidebar:
    name: Day 6
    identifier: aoc-2020-d6
    parent: Advent of Code 2020
    weight: 10
---

## Intro
We're dealing with customs forms today. Luckily, they're of a rather simple format: each line represents a passenger, and each letter on that line represents a question to which they answered 'yes' to. Passengers are grouped together, separated in the input by an empty line.
```
abcx
abcy
abcz
```
In this example, there is one group of 3 passengers, where there are 6 questions to which anyone answered "yes": a, b, c, x, y, and z.

## Solution
Since the groups can consist of multiple lines, each group is separated by an empty line. This requires us to split on `\n\n` again:

```kotlin
val input = rawInput.split("\n\n")
```

### Part 1
The first challenge is to determine the number of unique questions per group, and total these up over all groups. Again, this is rather easy: per group, put everything on the same line (`String.replace("\n", "")`), get the underlying array of characters, transform this to a set (since a set will filter out duplicates) and count the amount of items in this set. When we do this in a `List.map()`-call, we can then simply sum up all values with the `.sum()` method.

```kotlin
override fun part1(): String {
	return input.map {
		it.replace("\n", "").toCharArray().toSet().count()
	}.sum().toString()
}
```

### Part 2
The second challenge is not too difficult either: determine the amount of questions to which *everyone* answered yes per group, and again total these up. Given a group (called `singleInput` in the code): split up this group into its different lines (remember, these represent different passengers). Map these lines to the set of unique characters per line in a similar way like we did in part one (by retrieving its char-array and calling `.toHashSet()` on it. Note: we're calling `.toHashSet()` rather than just `toSet()` as we'll need access to the `.retainAll()` method later on, which is not available in the basic `Set` type). 

Now we have a stream of character-sets, which we can reduce to one set of common characters. We do this by calling `.reduce { acc, curSet -> acc.retainAll(curset); acc }`. Here, `acc` represents the accumulator, which is initialized to the first set in in the stream and updates accordingly through the call to `.retainAll()`. The `; acc` at the end is required as a reduce-call expects a value to be returned. Since we update `acc` through the `.retainAll()`-call, we can simply return `acc` here as it's already the updated version at this point. 

Once we have this set, call `.size` on it, and sum it all up.

```kotlin
override fun part2(): String {
	return input.map { singleInput ->
		singleInput.split("\n")
				.map { line -> line.toCharArray().toHashSet() }
				.reduce { acc, curSet -> acc.retainAll(curSet); acc }.size
	}.sum().toString()
}
```
