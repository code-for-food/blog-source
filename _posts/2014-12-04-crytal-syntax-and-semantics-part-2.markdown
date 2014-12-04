---
layout: post
title: "Crytal: Syntax and semantics Part 2"
date: 2014-12-04 16:46:54 +0700
comments: true
categories: 
---
###5.Control expressions
####5.1.Truthy and falsey values
A truthy value is a value that is considered true for an if, unless, while or until guard. A falsey value is a value that is considered false in those places.

The only falsey values are nil, false and null pointers (pointers whose memory address is zero). Any other value is truthy.
####5.2.if
An if evaluates the then branch if its condition is truthy, and evaluates the else branch, if there’s any, otherwise.
```
a = 1
if a > 0
  a = 10
end
a #=> 10

b = 1
if b > 2
  b = 10
else
  b = 20
end
b #=> 20
```
To write a chain of if-else-if you use *elsif*:
```
if some_condition
  do_something
elsif some_other_condition
  do_something_else
else
  do_that
end
```
After an *if*, a variable’s type depends on the type of the expressions used in both branches.
```
a = 1
if some_condition
  a = "hello"
else
  a = true
end
# a :: String | Bool

b = 1
if some_condition
  b = "hello"
end
# b :: Int32 | String

if some_condition
  c = 1
else
  c = "hello"
end
# c :: Int32 | String

if some_condition
  d = 1
end
# d :: Int32 | Nil
```
Note that if a variable is declared inside one of the branches but not in the other one, at the end of the *if* it will also contain the *Nil* type.

Inside an *if*'s branch the type of a variable is the one it got assigned in that branch, or the one that it had before the branch if it was not reassigned:
```
a = 1
if some_condition
  a = "hello"
  # a :: String
  a.length
end
# a :: String | Int32
```
That is, a variable’s type is the type of the last expression(s) assigned to it.

If one of the branches never reaches past the end of an *if*, like in the case of a *return*, *next*, *break* or *raise*, that type is not considered at the end of the *if*:
```
if some_condition
  e = 1
else
  e = "hello"
  # e :: String
  return
end
# e :: Int32
```
####5.3.unless
An *unless* evaluates the then branch if its condition is falsey, and evaluates the *else branch*, if there’s any, otherwise. That is, it behaves in the opposite way of an *if*:
```
unless some_condition
  then_expression
else
  else_expression
end

# The above is the same as:
if some_condition
  else_expression
else
  then_expression
end

# Can also be written as a suffix
close_door unless door_closed?
```
####5.4.case
A *case* is a control expression that allows a sort of pattern matching. It allows writing a chain of if-else-if with a small change in semantic and some more powerful constructs.

In its basic form, it allows matching a value against other values:
```
ase exp
when value1, value2
  do_something
when value3
  do_something_else
else
  do_another_thing
end

# The above is the same as:
tmp = exp
if value1 === tmp || value2 === tmp
  do_something
elsif value3 === tmp
  do_something_else
else
  do_another_thing
end
```
Note that *===* is used for comparing an expression against a *case*'s value.

If a *when*'s expression is a type, *is_a*? is used. Additionally, if the case expression is a variable or a variable assignment the type of the variable is restricted:
```
case var
when String
  # var :: String
  do_something
when Int32
  # var :: Int32
  do_something_else
else
  # here var is neither a String nor an Int32
  do_another_thing
end

# The above is the same as:
if var.is_a?(String)
  do_something
elsif var.is_a?(Int32)
  do_something_else
else
  do_another_thing
end
```
You can invoke a method on the *case*'s expression in a *when* by using the implicit-object syntax:
```
case num
when .even?
  do_something
when .odd?
  do_something_else
end

# The above is the same as:
tmp = num
if tmp.even?
  do_something
elsif tmp.odd?
  do_something_else
end
```
Finally, you can ommit the *case*'s value:
```
case
when cond1, cond2
  do_something
when cond3
  do_something_else
end

# The above is the same as:
if cond1 || cond2
  do_something
elsif cond3
  do_something_else
end
```
This sometimes leads to code that is more natural to read.
####5.5.while
A *while* executes its body as long as its condition is truthy
```
while some_condition
  do_this
end
```
In the above form, the condition is first tested and, if truthy, the body is executed. That is, the body might never be executed.

To execute the body at least once and then continue executing it while the condition is truthy, write the while as a prefix:v
```
do_this while some_condition
```
If you need to execute multiple expressions, group them between *begin* and *end*:
```
begin
  do_this
  do_that
end while some_condition
```
A while's type is always Nil.

Similar to an if, if a while's condition is a variable, the variable is guaranteed to not be nil inside the body. If the condition is an var.is_a?(Type) test, var is guaranteed to be of type Type inside the body. And if the condition is a var.responds_to?(:method), var is guaranteed to be of a type that responds to that method.

The type of a variable after a while depends on the type it had before the while and the type it had before leaving the while's body:
```
a = 1
while some_condition
  # a :: Int32 | String
  a = "hello"
  # a :: String
  a.length
end
# a :: Int32 | String
```
####5.6.until
An *until* executes its body until its condition is truthy. An until is just syntax sugar for a while with the condition negated:
```
until some_condition
  do_this
end

# The above is the same as:
while !some_condition
  do_this
end
```
An *until* can also be used in its suffix form, causing the body to be executed at least once. break and next can also be used inside an until.
####5.7.&&
An *&&* (and) evaluates its left hand side. If its truthy, it evaluates its right hand side and has that value. Otherwise it has the value of the left hand side. Its type is the union of the types of both sides.

You can think an *&&* as syntax sugar of an *if*:
```
some_exp1 && some_exp2

# The above is the same as:
tmp = some_exp1
if tmp
  some_exp2
else
  tmp
end
```
####5.8.||
An *||* (or) evaluates its left hand side. If its falsey, it evaluates its right hand side and has that value. Otherwise it has the value of the left hand side. Its type is the union of the types of both sides.

You can think an *||* as syntax sugar of an *if*:
```
some_exp1 || some_exp2

# The above is the same as:
tmp = some_exp1
if tmp
  tmp
else
  some_exp2
end
```
I will present about types and methods in part 3.
Thanks!

