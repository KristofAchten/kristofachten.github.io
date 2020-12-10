---
title: "Day 10: Adapter Array"
date: 2020-12-10T12:06:25+01:00
hero: /posts/Advent of Code 2020/aoc.jpeg
description: Day 10 of AoC 2020
menu:
  sidebar:
    name: Day 10
    identifier: aoc-2020-d10
    parent: Advent of Code 2020
    weight: 10
---

## Intro
Today turned out to be one of the more difficult days thus far, but the good news is that we get to dive into a new paradigm: [Dynamic programming!](https://en.wikipedia.org/wiki/Dynamic_programming)

Our input is a list of numbers, where each number represents the output *joltage* for an adapter. Each adapter can produce this output joltage, given that its input joltage is less than 4 units lower than its output joltage.

```
16
10
15
5
1
11
7
19
6
12
4
```
Other than that, the device that you're plugging in to has an adapter built in that produces a joltage of 3 jolts higher than the highest-rated adapter in the input-list. It is also assumed that the charging outlet that you use has an effective joltage rating of 0.

## Solution

As usual, we start by splitting up our input into its different lines. Since they're numbers, we immediately map them to longs (Int would be fine as well). 
Since it will prove useful in the challenges, we'll immediately sort this list, and add the 0-jolt input from the charging outlet as well as the joltage of the adapter that's built into the device.
```kotlin 
var input: ArrayList<Long> by Delegates.notNull()

init {
    val sortedInputList = rawInput.split("\n").map { it.toLong() }.sorted()

    input = arrayListOf<Long>(0) // Charging outlet
    input.addAll(sortedInputList)
    input.add(sortedInputList.max()!! + 3) // Device
}
```
### Part 1
The first challenge requires us to chain up all adapters in our input list, and determine the joltage differences between the devices in the chain. From these differences, we're supposed to multiply the amount of 1-diffs with the amount of 3-diffs.
Since our input is already sorted, this is easy as the list-class has a function that can do this for us: `List.zipWithNext()`. This function is similar to a fold-function, and could be used interchangeably.

```kotlin
val listOfDifs = input.zipWithNext { a, b -> (b - a).toInt() }
```
Once we have this list of differences, simply filter out the necessary value and count the size of the remaining list. We do this for both values 1 and 3 and multiply them together:

```kotlin
override fun part1(): String {
    return (listOfDifs.filter { it == 1 }.count() * listOfDifs.filter { it == 3 }.count()).toString()
}
```

### Part 2
For part two, we'll need to determine the number of possible ways in which we can arrange the adapters to form a valid adapter chain. Remember that an adapter can produce its output only if the difference with its input is smaller than 4.
To solve this challenge, we can use a paradigm called dynamic programming, where we determine the value per step, save it in an array of results, and use this array for the further calculations. This has an obvious benefit over recursion, as the same calculations will only have to occur once rather than on every equivalent run.

The implemented algorithm builds on the following foundations:
* The amount of possible arrangements between the power outlet and the first adapter is exactly 1
  * This means that we can insert the value `1` for the first two elements (being the 0 input and the first adapter) in our DP-array
* We continue down the list: at each possible element, we'll look back at the DP-array that we've constructed while knowing the following:
  * As no adapter is the same and we're dealing with discrete joltage-steps of 1, the possible alterations in arrangement start max 3 adapters back
  * For each of these three adapters, the alteration can only occur if the difference with the output-joltage of the current adapter is less than 3 (per definition)
  * The number of paths up to the current point is equal to the sum of the possible paths at the alteration point(s)
    * This essentially means: if alteration is possible at a (set of) given adapters max 3 places back, the amount of paths at the current point is the sum of the values that we've already determined for those adapters within our DP-array

Of course, to finalize, we update our DP-array during each step with the newly found value. To know the number of possible ways that we can arrange all adapters, we simply get the last number in our DP-array.

Algorithmically, this would look something like this (note: instead of a DP-array, I went for a list-type called `cache`)

```kotlin
  override fun part2() = countPaths().toString()

  fun countPaths(): Long {
      val cache = arrayListOf<Long>(1, 1)

      for (i in 2 until input.size) {
          val toAdd = listOf<Int>(1, 2, 3).map { j ->
              if (i > j - 1 && input[i] - input[i - j] <= 3) cache[i - j] else 0
          }.sum().toLong()

          cache.add(toAdd)
      }

      return cache[input.size - 1]
  }
```
    
  


