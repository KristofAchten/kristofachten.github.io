---
title: "Day 5: Binary Boarding"
date: 2020-12-05T12:06:25+01:00
hero: /posts/Advent of Code 2020/aoc.jpeg
description: Day 5 of AoC 2020
menu:
  sidebar:
    name: Day 5
    identifier: aoc-2020-d5
    parent: Advent of Code 2020
    weight: 10
---

## Intro
Apparently, we're boarding a plane today. Surely toboggans can fly, no? Especially when Santa's the one riding it. Anyway, today's input is a list of [Welsh words](https://www.reddit.com/r/adventofcode/comments/k737j6/day_5_welsh_booking_system/) :-). Each line looks as follows:

```
BFFFBBFRRR
```

B stands for 'Back', F stands for 'Front', L stands for 'Left', and R for 'Right. There are 128 rows and 8 colums in the plane. Given this initial configuration:
- F and B specify which row you're in on the plane
	- F means to take the lower half, keeping rows 0 through 63.
	- B means to take the upper half, keeping rows 64 through 127.
- L and R specify which column you're in on the plane
	- L means to take the lower half, keeping rows 0 through 3.
	- R means to take the upper half, keeping rows 4 through 7.

The entire sequence would then look something like this:
- Start by considering the whole range, columns 0 through 7.
- R means to take the upper half, keeping columns 4 through 7.
- L means to take the lower half, keeping columns 4 through 5.
- The final R keeps the upper of the two, column 5.

## Solution
We again start by processing or input. This time, it's simple: just split on `\n`;
```kotlin
val input = rawInput.split("\n")
```

### Part 1
For part one, we need to identify the highest possible seatID on the list. Such an ID is defined as `row * 8 + column`. Knowing this, a good way to start is by mapping all our input sequences to their respective ID.

There are multiple ways to tackle this solution: you could, for example, loop over each character in the sequence and use a `when`-statement to update the positional state according to the character that is read. A more clever solution is to read the title of this challenge, and realise that these sequences are actually just a masked form of binary! Don't worry, a colleague had to tell me too before I realized. 

By taking the sequence, replacing all characters `F` and `L` ('lower' half) with `0` and all characters `B` and `R` ('upper' half) with `1`, then splitting it up in the row and colums parts (`String.substring(0, 7)` and `String.substring(7)` respectively), you can simply parse each part to its respective value by calling `.toInt(2)` (where `2` is the radix, indicating that we're paring binary). Take the example above, where we determined the row for `RLR` to be 5. Using the defined tranformation, `RLR` becomes `101`, and `101 base 2` == `5 base 10`.

Having these values for row and column, the mapping of the sequences becomes easy as we just need to fill in the given formula.

Once this is done, to finish part one, we sort this list of IDs and take the last value of it. This is the highest ID on our input list.

```kotlin
val ids = input.map {
	val binSeq = it.replace("(F|L)".toRegex(), "0").replace("(B|R)".toRegex(), "1")
	binSeq.substring(0, 7).toInt(2) * 8 + binSeq.substring(7).toInt(2)
}.toList().sorted()

override fun part1() = ids[ids.size - 1].toString()
```

### Part 2
For part two, we'll need to find the missing ID in the list. As we've already sorted the list of IDs in part one, this is not that difficult: just loop over the indices of that list and check whether or not `ids[i] + 1 != ids[i + 1]`. If that *is* the case, the gap has been found and our solution is simply the value for that index + 1.

```kotlin
override fun part2() = (ids.get(ids.indices.find { i -> ids[i] + 1 != ids[i + 1] }!!) + 1).toString()
```


