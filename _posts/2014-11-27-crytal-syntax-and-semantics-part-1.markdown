---
layout: post
title: "Crytal: Syntax and semantics Part 1"
date: 2014-11-27 17:05:09 +0700
comments: true
categories: 
---

###1. Comments
```
# This is a comment
```
###2.Local variables
Local variables start with lowercase letters. They are declared when you first assign them a value.
```
name = "Crystal"
age = 1
```
Their type is inferred from their usage, not only from their initializer. In general, they are just value holders associated with the type that the programmer expects them to have according to their location and usage on the program.

For example, reassinging a variable with a different expression makes it have that expression’s type:
```
var = "Hello"
# At this point 'var' is a String

var = 1
# At this point 'var' is an Int32
```
Underscores are allowed at the beginning of a variable name, but these names are reserved for the compiler, so their use is not recommended (and it also makes the code uglier to read).
###3.Global variables
Global variables start with a dollar sign ($). They are declared when you first assign them a value.
```
$year = 2014
```
Their type is the combined type of all expressions that were assigned to them. Additionally, if you program reads a global variable before it was ever assigned a value it will also have the Nil type.
###4.Assignment
Assignment is done with the equal (=) character.
```
# Assigns to a local variable
local = 1

# Assigns to an instance variable
@instance = 2

# Assigns to a class variable
@@class = 3

# Assigns to a global variable
$global = 4
```
Some syntax sugar that contains the = character is available:
```
local += 1  # same as: local = local + 1

# The above is valid with these operators:
# +, -, *, /, %, |, &, ^, **, <<, >>

local ||= 1 # same as: local || (local = 1)
local &&= 1 # same as: local && (local = 1)
```
A method invocation that ends with = has syntax sugar:
```
# A setter
person.name=("John")

# The above can be written as:
person.name = "John"

# An indexed assignment
objects.[]=(2, 3)

# The above can be written as:
objects[2] = 3

# Not assignment-related, but also syntax sugar:
objects.[](2, 3)

# The above can be written as:
objects[2, 3]
```
The = operator syntax sugar is also available to setters and indexers. Note that || and && use the []? method to check for key prescence.
```
person.age += 1        # same as: person.age = person.age + 1

person.name ||= "John" # same as: person.name || (person.name = "John")
person.name &&= "John" # same as: person.name && (person.name = "John")

objects[1] += 2        # same as: objects[1] = objects[1] + 2

objects[1] ||= 2       # same as: objects[1]? || (objects[1] = 2)
objects[1] &&= 2       # same as: objects[1]? && (objects[1] = 2)
```
I will present about control expressions in part 2.
Thanks!

