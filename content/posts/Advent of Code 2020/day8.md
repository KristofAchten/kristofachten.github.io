---
title: "Day 8: Halting"
date: 2020-12-08T12:06:25+01:00
hero: /posts/Advent of Code 2020/aoc.jpeg
description: Day 8 of AoC 2020
menu:
  sidebar:
    name: Day 8
    identifier: aoc-2020-d8
    parent: Advent of Code 2020
    weight: 10
---

## Intro
[IntCode vibes intesify...](https://esolangs.org/wiki/Intcode) Today we're dealing with a simple programming language (INSIDE a programming language). There are only 3 instructions:
- `acc`: increases or decreases a global accumulator value
- `jmp`: jumps to a given instruction (relative to itself)
- `nop`: does nothing, is a no-op.

As an example, a program could look something like this:
```kotlin
nop +0
acc +1
jmp +4
acc +3
jmp -3
acc -99
acc +1
jmp -4
acc +6
```

## Solution
We start by parsing our input. Essentially, each program consists of multiple instructions, and each instruction consists of an operation ('acc', 'nop' or 'jmp) and an argument. I decided to define my own data class for instructions:
```kotlin
data class Instruction(var operation: String, val argument: Int, var ran: Boolean)
```
As you can see, an Instruction-instance also consists of a boolean value called `ran`. This will be used later on in the challenges to determine if an instruction has been ran before, and thus indicates the fact that there is a loop in the program.

We simply parse the input by splitting it up into its different lines, and mapping each such line to an `Instruction`-instance, storing all of these instances in a global list `runList`.
```kotlin
val runList = rawInput.split("\n").map { Instruction(it.split(' ')[0], it.split(' ')[1].toInt(), false) }.toList()
```

### Part 1
For part one, the challenge is to determine what the value is of the global accumulator variable just before we run an instruction twice. The function I've written for this is pretty straightforward, so I'm not going to discuss it in depth, but instead enriched my code with commentary where necessary.

```kotlin
private fun runProgram(): Int {
	var acc = 0 // This is the global accumulator variable
	var ctr = 0 // This is the program pointer, which indicates which instruction we're running in the runList

	while (true) {
		// Load in the instruction
		val instruction = runList.get(ctr)
		
		// If the loaded instruction has already executed once, terminate and return acc
		if (instruction.ran) {
			return acc
		}
		
		// Do something based on the operation and argument within the instruction
		when (instruction.operation) {
			"acc" -> {
				acc += instruction.argument; ctr++
			}
			"jmp" -> ctr += instruction.argument;
			"nop" -> ctr++
		}
		
		// Set the ran-value to indicate that this instruction has been ran
		instruction.ran = true
	}
}
```

Solving challenge one simply comes down to running the following code:
```kotlin
override fun part1() = runProgram().toString()
```

### Part 2
In part two, we're told that switching around one 'jmp' instruction with a 'nop' instruction or vice versa, will result in a terminating program. The question: what is the value of the accumulator once it does terminate?

To start of, let's refactor our solution for part one so that it can be used for part two. Since we're now no longer only terminating when a loop is detected, we'll need a notion of an exit-type for the function `runProgram()`. There are two such types: loop detected, and terminated succesfully. For easy referencing, I defined an enum for this:

```kotlin
enum class ReturnType { LOOP, SUCCESS }
```

Of course, we'll also still need to return the accumulator-value. Since kotlin does not support multiple return values (like GoLang does for example), we'll need to wrap this in a data class:

```kotlin
data class ProgramResult(val resultType: ReturnType, val resultAcc: Int)
```

The challenge states that a solution is terminating when it references an instruction that is out of bounds of the `runList`. We simply need to add a condition that checks this.

Combining all of this, we get the following definition for `runProgram()`:

```kotlin
private fun runProgram(): ProgramResult {
	var acc = 0
	var ctr = 0

	while (true) {
		val instruction = runList.get(ctr)
		if (instruction.ran) {
			return ProgramResult(ReturnType.LOOP, acc)
		}
		when (instruction.operation) {
			"acc" -> {
				acc += instruction.argument; ctr++
			}
			"jmp" -> ctr += instruction.argument;
			"nop" -> ctr++
		}

		// If we're out of bounds, terminate succesfully
		if (ctr < 0 || ctr >= runList.size) {
			return ProgramResult(ReturnType.SUCCESS, acc)
		}

		instruction.ran = true
	}
}
```

Part one can then be altered as follows:

```kotlin
override fun part1() = runProgram().resultAcc.toString()
```

For part two, my code is (again) pretty straightforward: we simply loop through `runList`: if we encounter a 'jmp', we replace it with 'nop' and vice versa. Then we run the program, check if it is terminating and return the accumulator value if it is. If it's not, we simply reset the instruction, continue through the list, remembering to reset all our `ran`-values to false (otherwise, false loops would be detected).

```kotlin
private fun resetInstructions() {
	runList.forEach { it.ran = false }
}

private fun findTerminatingSolution(): Int {
	runList.forEach { instruction ->
		resetInstructions()

		var oldOp = instruction.operation
		when (oldOp) {
			"jmp" -> instruction.operation = "nop"
			"nop" -> instruction.operation = "jmp"
		}

		val result = runProgram()
		if (result.resultType == ReturnType.SUCCESS) {
			return result.resultAcc
		}

		instruction.operation = oldOp
	}

	throw IllegalStateException("Could not fix the given program so that it terminates.")
}
```

Having these function definitions, solving part two can be done by calling `findTerminatingSolution()`:

```kotlin
override fun part2() = findTerminatingSolution().toString()
```