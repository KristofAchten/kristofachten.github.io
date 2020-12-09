---
title: "Day 7: Bags"
date: 2020-12-07T12:06:25+01:00
hero: /posts/Advent of Code 2020/aoc.jpeg
description: Day 7 of AoC 2020
menu:
  sidebar:
    name: Day 7
    identifier: aoc-2020-d7
    parent: Advent of Code 2020
    weight: 10
---

## Intro
Interesting challenge today: recursion! We've landed at the airport and get to take on the exciting challenge of retrieving our bags from the carousel of doom. Fortunately, bags are color coded. Unfortunately, the bags can be inside each other with respect to a set of predefined regulations. Our puzzle input is this set of regulations, which looks a bit like this:

```kotlin  
light red bags contain 1 bright white bag, 2 muted yellow bags.
dark orange bags contain 3 bright white bags, 4 muted yellow bags.
bright white bags contain 1 shiny gold bag.
muted yellow bags contain 2 shiny gold bags, 9 faded blue bags.
shiny gold bags contain 1 dark olive bag, 2 vibrant plum bags.
dark olive bags contain 3 faded blue bags, 4 dotted black bags.
vibrant plum bags contain 5 faded blue bags, 6 dotted black bags.
faded blue bags contain no other bags.
dotted black bags contain no other bags.
```

These rules specify 9 bag types (= colors) and their contents.


## Solution
As you might've expected, parsing our input is a challenge on its own. There are a few ways to tackle this, but since they all follow roughly the same pattern, I simple went with extensively using `String.split()`.

First however, we'll need to think about a way to store these bags. The chances are high that we'll effectively need to search what the contents of a bag are given its color, so it makes sense to use a Map-type to store them in as this allows for easy querying. Given a color, the map should return a list of tuples containing the contents of that bag. 

The map could look something like this: `Map<color: String, content: List<Pair<color: String, count: Int>>>`. Just for clarity, I decided to put `Pair<color: String, count: Int>` in its own data definition:

```kotlin
data class BagContent(val color: String, val count: Int)

val bagDefs = mutableMapOf<String, List<BagContent>>()
```

For the actual processing of the input, we start by splitting it up into its different lines:
```kotlin
val input = rawInput.split("\n")
```

Given each line, we can then use its somewhat steady pattern to split the definition up into the different parts. 

First, we'll need to separate its color from its content, which can easily be done through calling `line.split(" bags contain ")` (note the spaces at the start and at the end). Then, continue by splitting up the contents... but only if it has any to start with! Check that `content.equals("no other bags.")` returns false before you continue. Remove the dots and comma's from the remaining string and replace the word 'bags' by 'bag' (since they will be used interchangeably depending on the count for that type of bag). This now allows us to split on the word `bag` using `.split(" bag")` (again, mind the space!), but this has the tendency to generate an empty string at the end of the resulting list since `bag` is the last word in the original string. This is just how `.split()` works ([and this is why](https://stackoverflow.com/questions/4964484/why-does-split-on-an-empty-string-return-a-non-empty-array)), so just filter them out using `.filter { it.isNotBlank() }`.

At this point, we have a list of content definitions that are separated by spaces. Simply split each of them up using `.split(" ")`, and add them to a `List<BagContent>`. If your bag did not have any contents to begin with, just leave this list empty.

Finally, just put the color and this generated list of `BagContent` data in the `bagDefs`-map.

```kotlin
init {
	for (line in input) {
		val bagAndContent = line.split(" bags contain ")

		var contentList = mutableListOf<BagContent>()
		if (!bagAndContent[1].equals("no other bags.")) {
			val contentDefs = bagAndContent[1]
					.replace(".", "")
					.replace(", ", "")
					.replace("bags", "bag")
					.split(" bag")
					.filter { it.isNotBlank() }

			contentDefs.forEach() { contentDef ->
				val parts = contentDef.split(" ")
				contentList.add(BagContent(parts[1] + " " + parts[2], parts[0].toInt()))
			}
		}

		bagDefs.put(bagAndContent[0], contentList)
	}
}
```
### Part 1
For part one, the question at hand is straightforward: How many bag colors can eventually contain at least one shiny gold bag?

Essentially, this requires us to go through all the definitions we just inserted into `bagDefs`, and filter out those where there is no path towards a `shiny gold` bag. As soon as we have this filtered list, we just return the amount of elements in it.

To determine if a bag can (eventually) contain the shiny gold bag, we'll need to define a recursive function. This function will need to stop (and return true) if it gets to a bag where the color is `shiny gold`, or else check if any of its content-bags could contain this color. This gives use the following definition:

```kotlin
private fun containsBag(color: String, curColor: String, list: List<BagContent>): Boolean {
	if (color.equals(curColor)) {
		return true
	}
	return list.any() { containsBag(color, it.color, bagDefs.get(it.color)!!) }
}
```

Given this function, solving part one becomes trivial. One thing we'll need to account for is the fact that the shiny gold bag itself will return true as well (for obvious reasons), but for this challenge we're not supposed to count it. We can easily extend our filter to not include this bag definition.

```kotlin
val shinyGold = "shiny gold"

override fun part1() = bagDefs.filter { !it.key.equals(shinyGold) && containsBag(shinyGold, it.key, it.value) }.count().toString()
```

### Part 2
Part two is similar in nature, but the question is different: How many individual bags are required inside your single shiny gold bag?

So essentially: what is the total number of bags (or if you prefer: nodes) in the tree that is defined by the shiny gold bag? Again, we can define a recursive function for this. However, since we're now actually counting something in this function, the definition becomes slightly more difficult: given a color, the function will look up the contents for it in our `bagDefs`-map. If this list of contents is empty, it will simply stop and return `0`. If it is not empty: it will return the sum of the results for the recursive calls on each of the bags in its content list.

```kotlin
return bagDefs.get(color)!!.map { it.count * (1 + countBags(it.color)) }.sum()
```
This should be relatively straightforward: you determine the amount of bags in a given color, and multiply it by the amount of bags that were there in the first place. An easy mistake to make is forget about the current bag itself, and thus omitting the `1 +` (or alternatively: `it.count + it.count * countBags(it.color)`) from the statement above.

This gives us our final result:

```kotlin
override fun part2() = countBags(shinyGold).toString()

private fun countBags(color: String): Int {
	if (bagDefs.get(color)!!.isEmpty()) {
		return 0
	}
	return bagDefs.get(color)!!.map { it.count * (1 + countBags(it.color)) }.sum()
}
```