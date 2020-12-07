---
title: "Day 1: Expense report"
date: 2020-12-01T12:06:25+01:00
hero: /posts/Advent of Code 2020/aoc.jpeg
description: Day 1 of AoC 2020
menu:
  sidebar:
    name: Day 1
    identifier: aoc-2020-d1
    parent: Advent of Code 2020
    weight: 10
---

## Intro
It's that time of the year again: the time where we help Santa and his elves by solving... well... programming puzzles? The use of it may be debatable, but it sure is a lot of fun if you're into programming!

This year, I had some trouble deciding which programming language to use for the challenges. My colleagues suggested Kotlin since it inherently just is an awesome language, but as I'm currently following an introductory course to Angular, it wouldn't be a stupid decision to go for something like TypeScript or JavaScript either. The internal debate went on, but I finally decided on Kotlin as the main project language, with TypeScript being a secondary alternative if at some point I feel like solving the same puzzle twice.

## Setup

With the decision making done, I set up a simple project structure using IntelliJ and pushed it to a newly created [github repository](https://github.com/KristofAchten/AoC2020). In this structure, there are only two noteworthy class definitions:

- `abstract class Puzzle(day: Int)`: This is the class that will be implemented by the daily challenges. It's been setup to automatically fetch my personalized puzzle input from [adventofcode.com](https://adventofcode.com/) through a simple HTTP GET request in its constructor (well, *actually* in its `init` block), and contains two abstract function definitions (one for each part of the challenge) to be implemented by the caller. Other than this, it contains a `solve()` method that calls the two abstract functions and nicely prints its results out to the output stream.
```kotlin
abstract class Puzzle(day: Int) {

    var rawInput : String by Delegates.notNull()
    var day : Int by Delegates.notNull()

    init {
        val challengeUrl = URL("https://adventofcode.com/2020/day/$day/input")
        val sessionKey = File("kotlin/src/sessionkey.txt").readText()
        val connection: HttpURLConnection = challengeUrl.openConnection() as HttpURLConnection

        connection.setRequestProperty("Cookie", "session=$sessionKey")

        this.rawInput = connection.inputStream.bufferedReader().readLines().stream().collect(Collectors.joining("\n"))
        this.day = day
    }

    open fun solve() = "Solutions day ${day}: part 1 -> `${part1()}`, part 2 -> `${part2()}`"

    abstract fun part1() : String
    abstract fun part2() : String
}
```
- `main class`: Contains a list of class-instantiations over which it will loop and call the `solve()` method. It will then log all solutions to the README.md file, which allows for a nice representation of the found solutions on the main page of my [github repository](https://github.com/KristofAchten/AoC2020).
```kotlin
private val puzzles = mutableListOf<Puzzle>(
        D1ExpenseReport(), D2Passwords(), D3Slopes(), D4Passports(),
        D5Boarding(), D6CustomForms()
)

fun main() {
    val file = File("README.md")
    val sb = StringBuilder()

    for (p in puzzles) {
        var result = ""
        val execTime = measureTimeMillis {
            result = p.solve();
        }
        sb.append("- " + result + " *(execution took ${execTime}ms)*\n")
        println(result)
    }

    file.writeText(sb.toString())
}
```

## Solution

### Part 1
So the first puzzle was as easy as you can expect it to be: given a list of numbers seperated by the newline-character, find the two numbers that sum up to `2020` and multiply them together. We start be separating the input.

```kotlin
val input = rawInput.split('\n')
```

Since it's only day one and at this point I know nothing about Kotlin, I decided to go with the simple solution and just loop over the numbers, determining its difference with `2020` and checking whether or not the resulting number was present in the provided input. If this is the case: multiply and return. Done.

```kotlin
override fun part1(): String = findAndMultiply2Components(2020).toString()

fun findAndMultiply2Components(totalVal: Int): Int {
	for (i in input) {
		if (input.contains((totalVal - i.toInt()).toString())) {
			return i.toInt() * (totalVal - i.toInt())
		}
	}
	return 0
} 
```

### Part 2
Almost identical to part one, except: this time we have to find 3 numbers that sum up to 2020. This time I simply went with three for-loops iterating over the input list, but only starting at the position of the previous index to avoid double work. I could've done this in two for loops (in analogy with my solution for part one), but there isn't necessarily a clear advantage of doing so. 

Code ran, numbers multiplied and second star GET :-).

```kotlin
override fun part2(): String = findAndMultiply3Components(2020).toString()

fun findAndMultiply3Components(totalVal: Int): Int {
	for (i in 0..input.size - 1) {
		for (j in i + 1..input.size - 1) {
			for (x in j + 1..input.size - 1) {
				if (input[i].toInt() + input[j].toInt() + input[x].toInt() == totalVal) {
					return input[i].toInt() * input[j].toInt() * input[x].toInt()
				}
			}
		}
	}
	return 0
}
```

Oh right! A small side note for future reading: I enjoy using brackets for clarity (and because of ). In the snippet above, the curly braces could've been omitted without impact.