---
title: "Day 3: Toboggan Trajectory"
date: 2020-12-03T12:06:25+01:00
hero: /posts/Advent of Code 2020/aoc.jpeg
description: Day 3 of AoC 2020
menu:
  sidebar:
    name: Day 3
    identifier: aoc-2020-d3
    parent: Advent of Code 2020
    weight: 10
---

## Intro
Let's get down to sleighing! Today, we're hopping on a [toboggan](https://en.wikipedia.org/wiki/Toboggan) and see how many trees we can fell.

Our input is 2D-map/layout provided as a data-matrix:
```javascript
X.##.......
#...#...#..
.#....#..#.
..#.#...#.#
.#...##..#.
..#.##.....
.#.#.#....#
.#........#
#.##...#...
#...##....#
.#..#...#.#
```

In this input, `#` represents a tree and `.` is just open space. `X` is our starting position.

The data pattern repeats itself to the right:

```javascript
.##.........##.........##.........##.........##.........##.......  --->
#...#...#..#...#...#..#...#...#..#...#...#..#...#...#..#...#...#..
.#....#..#..#....#..#..#....#..#..#....#..#..#....#..#..#....#..#.
..#.#...#.#..#.#...#.#..#.#...#.#..#.#...#.#..#.#...#.#..#.#...#.#
.#...##..#..#...##..#..#...##..#..#...##..#..#...##..#..#...##..#.
..#.##.......#.##.......#.##.......#.##.......#.##.......#.##.....  --->
.#.#.#....#.#.#.#....#.#.#.#....#.#.#.#....#.#.#.#....#.#.#.#....#
.#........#.#........#.#........#.#........#.#........#.#........#
#.##...#...#.##...#...#.##...#...#.##...#...#.##...#...#.##...#...
#...##....##...##....##...##....##...##....##...##....##...##....#
.#..#...#.#.#..#...#.#.#..#...#.#.#..#...#.#.#..#...#.#.#..#...#.#  --->
```

## Solution
Let's start by processing our input. Since we're dealing with a 2D-map, it makes sense to dump the input into a 2D array. Doing so isn't that difficult: just split the data in the different lines (using `\n` as the delimiter), and treat each line (which is a String) as an array of characters. At this point, your Y-coordinate is the index of the line you're on, and the X-coordinate is the index of the character within that line. Put that into an `Array<Array<String>>` object and you're done.

```kotlin
val input = create2DArray(rawInput.split('\n'))
val tree = "#"

private fun create2DArray(parts: List<String>): Array<Array<String>> {
	return Array(parts.size) { i ->
		Array(parts.get(i).length) { j ->
			parts.get(i)[j].toString()
		}
	}
}
```

### Part 1
For part one, we're told that the toboggan will be moving 3 positions to the right and 1 position down per timestep. For the remainder of this post, I will be calling these values `difX` and `difY`. Knowing this, we're supposed to determine how many trees (= `#`-characters) we meet while traveling down until we reach the bottom of the map. Remember that the map expands infinitely to the right. 

#### Updating the coordinates per step
Updating the Y-position is simple: we simply set it to `y += difY`. Updating the X-position (remember, this is the horizontal position) is trickier, as doing something similar like we did for Y would quickly result in an OOB-exeption. A naive solution to this would be to copy the map X amount of times, and stitch it to the `input` value. Luckily, there is no need to do this, as we have the mighty modulus operator `%` at our command. we can simply use the modulus operator to determine the correct value that lies within the bounds of the input array as follows: `x = (x + difX) % width`. In the code, this `width` will be represented by `input[0].size`, as this is simply the width of the first row of the input, which is fine as all rows in the input have the same width.  

Knowing the above, the algorithm to determine the amount of trees we've hit while traveling down becomes trivial. We maintain a simple state (`curX` and `curY` for the current coordinates, initialized at (0, 0), and `trees` as the count of trees that have been hit, initialized at 0). The state is updated in a while-loop that runs as long as we've not hit the bottom and are not going to go beyond the bottom in our next iteration, and updates the coordinates as described above on each run. Before that, we check if `input[curY][curX] == '#'` and increment `trees` if this is the case. In the end, we simple return the value of `trees`.

```kotlin
override fun part1() = determineTreesForSlope(3, 1).toString()

fun determineTreesForSlope(difX: Int, difY: Int) : Int {
	var curX = 0
	var curY = 0
	var trees = 0

	while (curY <= (input.size - difY)) {
		if (input[curY][curX] == tree) {
			trees++
		}
		curX = (curX + difX) % input[0].size
		curY += difY
	}

	return trees
}
```

### Part 2
For part two, we get more slopes to try out. For each slope, we need to determine the number of trees hit, and multiply all those values together. Since we've already defined a generic function `determineTreesForSlope` to handle this  calculation, this is easy. 

```kotlin
override fun part2(): String {
	val multTrees = determineTreesForSlope(1, 1) *
			determineTreesForSlope(3, 1) *
			determineTreesForSlope(5, 1) *
			determineTreesForSlope(7, 1) *
			determineTreesForSlope(1, 2)
	return multTrees.toString()
}
```
