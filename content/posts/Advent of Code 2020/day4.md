---
title: "Day 4: Passport Processing"
date: 2020-12-04T12:06:25+01:00
hero: /posts/Advent of Code 2020/aoc.jpeg
description: Day 4 of AoC 2020
menu:
  sidebar:
    name: Day 4
    identifier: aoc-2020-d4
    parent: Advent of Code 2020
    weight: 10
---

## Intro
Aah yes, [papers please!](https://papersplea.se/) Today, we're determining which passports get stamped and which do not! Our input is, to no surprise, a list of passports of which each passport is defined as follows:

```
iyr:2019
hcl:#602927 eyr:1967 hgt:170cm
ecl:brn pid:012533040 byr:1946
```

They are simple `key:value` pairs, seperated over multiple lines.

## Solution
As usual, we start by seperating our raw input into usable chunks. This time, each passport (because it can contain multiple lines) is separated by an empty line. We therefor split on the `\n\n` delimiter.
```kotlin
val input = rawInput.split("\n\n")
```

Now having each separate passport, we can extract the `key:value` pairs, and ideally stow them away in a map for easy access later. Extracting the pairs is easy: we first put everything on the same line by calling `String.replace('\n', ' ')` on the passport, then splitting on spaces `String.split(' ')`, and finally splitting on the colon character `String.split(':')`. We collect this as a map per passport, and then collect all these maps as a list which we dump in a global variable.

```kotlin
var kvSets: List<Map<String, String>> by Delegates.notNull()

init {
	this.kvSets = input.map {
		it.replace("\n", " ")
				.split(" ")
				.map() { Pair(it.split(":")[0], it.split(":")[1]) }
				.toMap()
	}.toList()
}
```

### Part 1
For part one, we have to determine which passports are valid. A passport is valid if it at least contains the mandatory fields (see code below). Checking this is easy given the list of maps we've just created: simply loop over each map in the list and map this map (hah..) to a boolean value by checking if there is any required field that's not in that map (which would yield true, and false if all of them are in the given map). In the end, we filter out all of the true-values and count the size of the remaining list. This is our answer.

```
val reqFields = listOf<String>("byr", "iyr", "eyr", "hgt", "hcl", "ecl", "pid")

override fun part1() = countValidPassports().toString()

private fun countValidPassports(): Int {
	return kvSets.map { kvSet -> 
		reqFields.any { !kvSet.containsKey(it)}
	}.filterNot { it }.count()
}
```

### Part 2
Part two is, again, trickier. Per key, we now have a rule that tells us whether or not its value is valid. Given these rules, determine all passports that are valid.

I decided it would be nice to define a separate decision function for each rule that determines whether the input value is valid. We can then simply transform `reqFields` into a map of type `Map<String, (String) -> Boolean>`. The key is... well, the key (like `'iyr'`, `'hcl'`, ...), and the value is the decision function. I will define the decision function for each key in the paragraphs below.

#### Decision function for 'byr', 'iyr' and 'eyr'
These are straightforward checks to see if their value lies between two constants.

```kotlin
val validByr: (String) -> Boolean = { data -> data.toInt() >= 1920 && data.toInt() <= 2002 }
val validIyr: (String) -> Boolean = { data -> data.toInt() >= 2010 && data.toInt() <= 2020 }
val validEyr: (String) -> Boolean = { data -> data.toInt() >= 2020 && data.toInt() <= 2030 }
```
#### Decision function for 'hcl'
Here, we check whether the value starts with `#`, and contains exactly six characters in the alphabet [a-f0-9]. This smells like regular expressions could be useful, and you're right!

```kotlin
val validHcl: (String) -> Boolean = { data -> data[0] == '#' && "^([a-f0-9]{6})*$".toRegex().matches(data.substring(1)) }
```

#### Decision function for 'ecl'
Again, fairly simple: just checking if the value is part of a predefined list of values.

```kotlin
val validEcl: (String) -> Boolean = { data -> listOf<String>("amb", "blu", "brn", "gry", "grn", "hzl", "oth").contains(data) }
```

#### Decision function for 'pid'
You'll need to check that the value consists of exactly 9 numbers. I cheated a little bit, and just checked that the length was 9. This worked out fine for me, but know that you can use another regex to make sure they're all numbers as well.

```kotlin
val validPid: (String) -> Boolean = { data -> data.length == 9 }
```

#### Decision function for 'hgt'
This is the one that requires the most code, but it's not necessarily more difficult. We simply determine (given that the value is longer than 2 characters long) the unit ('in' or 'cm'), and perform extra checks based on the rules per unit.

```kotlin
val validHgt: (String) -> Boolean = { data ->
	if (data.length > 2) {
		val value = data.substring(0, data.length - 2).toInt()
		val unit = data.substring(data.length - 2)

		if (unit == "cm") value >= 150 && value <= 193
		else if (unit == "in") value >= 59 && value <= 76
		else false
	} else false
}
```

#### Final setup

Having all of these decision functions, we can expand `reqFields`;

```kotlin
val reqFields = mapOf<String, (String) -> Boolean>(
		Pair("byr", validByr),
		Pair("iyr", validIyr),
		Pair("eyr", validEyr),
		Pair("hgt", validHgt),
		Pair("hcl", validHcl),
		Pair("ecl", validEcl),
		Pair("pid", validPid)
)
```

We now update the `countValidPassports()` method to account for these decision functions. To make sure that this function still works with part one, we add the `checkRestrictions` input parameter that determines whether or not the decision functions will be checked.

```kotlin
override fun part1() = countValidPassports(false).toString() // updated
override fun part2() = return countValidPassports(true).toString()

private fun countValidPassports(checkRestrictions: Boolean): Int {
	return kvSets.map { kvSet ->
		reqFields.any { field ->
			!kvSet.containsKey(field.key) || (checkRestrictions && !field.value(kvSet.getValue(field.key)))
		}
	}.filterNot { it }.count()
}
```


