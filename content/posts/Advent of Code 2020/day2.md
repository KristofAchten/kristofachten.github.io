---
title: "Day 2: Passwords"
date: 2020-12-02T12:06:25+01:00
hero: /posts/Advent of Code 2020/aoc.jpeg
description: Day 2 of AoC 2020
menu:
  sidebar:
    name: Day 2
    identifier: aoc-2020-d2
    parent: Advent of Code 2020
    weight: 10
---

## Intro
Today, we're dealing with passwords. Given a defined policy, we'll need to determine which passwords are valid with respect to that policy. The policies are defined as follows:
```
1-3 a: abcde
```

## Solution
Again, we start by splitting up our input:
```kotlin
val input = rawInput.split('\n')
```
### Part 1
For part 1, the policy is simple. Given the example above:
- `1` and `3` are the minimal and maximal occurences that the given character `a` can have in the string `abcde`
	- In this case, we're all good, because `a` appears only once and thus lies between [1, 3]
	- Given the input `2-3 d: abcd` - we would conclude that the password is invalid as `d` only appears once, which is not part of [2, 3]

To tackle this problem, I started by creating a generic higher-order function `findValidPwds()` which takes a decision-function (which decides whether or not a policy is met) as its input. 

It starts by looping over the different input-strings, parsing it each time and determining the different parts `min`, `max`, `c` (char) and `str` (string). Once it has done this, it will call the decision-function and add its result to an accumulator called `validPwds`. Once it's done, it simply returns the string-value of this accumulator.
```kotlin
private fun findValidPwds(compFunc: (Int, Int, Char, String) -> Int): String {
	var validPwds = 0
	for (def in input) {
		val parts = def.replace(":", "").split(" ")
		val minMax = parts[0].split("-")

		val min = minMax[0].toInt()
		val max = minMax[1].toInt()
		val chr = parts[1].single()
		val str = parts[2]

		validPwds += compFunc(min, max, chr, str)
	}

	return validPwds.toString()
}
```	

At this point, to solve part one, we simply have to define a decision function. This function should take `min`, `max`, `c` (char) and `str` (string) as input, and decide whether or not the policy is met by returning `1` = A-OK or `0` = NOPE. In this specific case, we can use a Regular Expression which encapsulates the given char `c`, and call the `findAll()` method on this regex with the provided string `str` as input. This will return the amount of occurences of `c` in `str`. We check if this number lies between `min` and `max`, and return `1` if this is the case, or `0` otherwise.  

Given this decision function, simply pass it on to `findValidPwds()` and we get the answer.
```kotlin
override fun part1(): String {
	return findValidPwds(
			fun(min: Int, max: Int, c: Char, s: String): Int {
				val pattern = Regex(c.toString())
				val cnt = pattern.findAll(s).count()

				return if (cnt >= min && cnt <= max) 1 else 0
			}
	);
}
```

### Part 2

For part two, the policy is a little bit trickier. The values for `min` and `max` in our decision function are no longer the minimal and maximal occurences of a given character in the string, but rather indexes within that string. For this reason, I will now call them `i1` and `i2`. **Important to note: these are not 0-based indexes when accessing the string!** As an example: `1-2 ...` would indicate characters 1 and 2 of a given string `str`, meaning `str[0]` and `str[1]`.

With this in mind, the policy takes the following form: a password is valid if *exactly one* of the characters as defined by `i1` and `i2` is equal to char `c`. The 'exactly one' is crucial here, but luckily Kotlin has built-in XOR-support for booleans, so the decisions function is fairly straightforward.

Another thing to note is that it is now possible to go out of bounds when accessing our string `str`. When taking `3-7 c: abcd` as an example, the policy holds because `str[3-1] == c`, but `str[7-1]` is undefined! In this specific example, because of [lazy evaluation](https://en.wikipedia.org/wiki/Lazy_evaluation), it wont cause any issues, but consider `5-8 c: abcd` or `4-7 c: abcd` and you will see why it is necessary for us to boundary checking when accessing the string `str`.

Knowing all of this, we get the following decision function. Plug it into `findValidPwds()` et voila: second star in the pocket!

```kotlin
override fun part2(): String {
	return findValidPwds(
			fun(min: Int, max: Int, c: Char, s: String): Int {
				val shiftMin = min - 1
				val shiftMax = max - 1

				return if ((shiftMin < s.length && s[shiftMin] == c)
								.xor(shiftMax < s.length && s[shiftMax] == c)) 1 else 0
			}
	);
}
```