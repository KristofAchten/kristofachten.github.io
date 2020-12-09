---
title: "Day 9: Encoding"
date: 2020-12-09T12:06:25+01:00
hero: /posts/Advent of Code 2020/aoc.jpeg
description: Day 9 of AoC 2020
menu:
  sidebar:
    name: Day 9
    identifier: aoc-2020-d9
    parent: Advent of Code 2020
    weight: 10
---

## Intro
Another day, another puzzle! It's time to deal with some form of simple encryption, but they're keeping it pretty light.

The puzzle input is a set of data encrypted by the XMAS-protocol. This protocol starts by transmitting a preamble of 25 numbers. After that, each number you receive should be the sum of any (unique) two of the 25 immediately previous numbers.

## Solution

We start by splitting up our data into lines, as each line shows a number. We'll do something different however, and immediately map the numbers to a long-type (don't use integers, some numbers won't fit!) rather than just a string. This will be useful in further solving of the challenges.

```kotlin
val input = rawInput.split("\n").map { it.toLong() }
```

### Part 1

Part one requires us to find the first error in the list, meaning the first number that is not a sum of any unique two of the previous 25 numbers. As you'll see, this is not too difficult.

We start off by running over our input. Since there is a preamble of size 25, we can ignore the first 25 numbers (as they are, per default, valid) and start at number 26. Per number, we do something similar as we did during [day 1](https://kristofachten.github.io/posts/advent-of-code-2020/day1/) and check if the subset of the previous 25 numbers contains two other numbers that sum up to it. For this, I created a separate function which, given the subset and a number, determines if for any of the numbers in the subset, the difference with the original number is also in that subset:

```kotlin
private fun pairExists(value: Long, subList: List<Long>) = subList.any {
	input.contains(value - it)
}
```

As soon as we find a number for which this is not true, we have found our error and can simply return it.

```kotlin
    val invalidNumber = findInvalidNumber(25)
	
    override fun part1() = invalidNumber.toString()

    private fun findInvalidNumber(preambleSize: Int): Long {
        for (i in preambleSize..input.size) {
            if (!pairExists(input.get(i), input.subList(i - preambleSize, i))) {
                return input.get(i)
            }
        }

        throw IllegalStateException("Could not determine an invalid number!")
    }

```

### Part 2
Part two continuous on part one, and requires us to find the first contiguous set of numbers which sums up to the error we found. From this set, we need to return the sum of the min- and max values.

The function for this is not that difficult. We loop over the indexes of `input`, each time treating the index as the starting index for the contiguous subset, and keep on adding numbers in order until we get to a value that is bigger or equal to the one that we're searching. Once we get there, we check if its actually a subset with more than one element (otherwise, a set only containing our error would also be valid) and that the accumulated sum is actually equal to the error. If this is the case, we extract the sublist and determine the min- and max values through `.min()` and `.max()` calls. Add them up and return.

```kotlin
override fun part2() = findContiguousSetAndSumMinMax(invalidNumber).toString()

private fun findContiguousSetAndSumMinMax(invalidNumber: Long): Long {
	for (i in input.indices) {
		var sum = 0L
		var curIdx = i

		while (curIdx < input.size && sum < invalidNumber) {
			sum += input.get(curIdx++)
		}

		if (i != curIdx - 1 && sum == invalidNumber) {
			val subList = input.subList(i, curIdx - 1)
			return subList.min()!! + subList.max()!!
		}
	}

	throw IllegalStateException("Could not find contiguous set!")
}
```