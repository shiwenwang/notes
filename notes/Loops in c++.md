---
tags: [C++]
title: Loops in c++
created: '2020-03-26T10:46:24.792Z'
modified: '2020-05-20T00:50:34.466Z'
---

# Loops in c++

A **While loop** will always evaluate the condition first.

```c++
while (condition)
{
  // doing  after condition is checked.
}
```

A **do/while loop** will always execute the code in the `do{}` block first and then evaluate the condition.

```c++
do 
{
	// gets executed at least once
} while (condition)
```

A **for loop** allows you to initiate a counter variable, a check condition, and a way to increment your counter all in one line.

```c++
for (int x=0; x<100; x++)
{
	// executed until x >= 100
}
```

At the end of the day, they are all still loops, but they offer some flexibility as to how they are executed.

Here is a great explanation of the *reasoning* behind the use f each different type of loop that may help clear thing up.
> The main difference between the `for`'s and the `while`'s is a matter of pragmatics(跟语境有关): we usually user `for` when there is a known number if iterations, and use `while` constructions when the number if iterations in not known in advance. The `while` vs `do...while` issue id also of pragmatics, the second executes the instructions once at start, and afterwards it behaves just like the simple while.
