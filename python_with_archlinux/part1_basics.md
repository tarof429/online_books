# Part 1: Installation and the Basics

## Installation

On ArchLinux, the *python* package will install Python 3.x.

```bash
$ pacman -Q python
python 3.8.6-1
$ python --version
Python 3.8.6
```

## Comments

To write a comment, prefix the line with a ‘#’ character. The line will not be executed.

```python
# defining the post code
postCode = 75000
```

Comments can span multiple lines and is often used at the beginning of the file. 

```python
# LinuxThingy version 1.6.5
#
# Parameters:
#
# -t (--text): show the text interface
# -h (--help): display this help
```

To write multi-line comments, instead of using the '#' character, you can use '''.

```python
"""
LinuxThingy version 1.6.5

Parameters:

-t (--text): show the text interface
-h (--help): display this help
"""
```

## Numbers and strings

Python can be used a simple calculator, as the examples below show.

``` python
# Returns 9
2 + 7

# -3
2 - 5

# 6
2 * 3

# Returns 2.0 which is a float by default
10 / 5 
    
# Returns 2 using integer division
10 // 5 
    
# Power function which returns 8
2 ** 3 
```

Python does not have a ++ operator like in C. To increment a number you must explicitly add 1 to it. The example below counts the number of characters in *world*.

```python
world = 'world'
counter = 1
for _, s in enumerate(world):
    counter = counter + 1

print(counter) # 6
```

Strings are ordered collections of characters. They can be single, double or triple quoted.

```python
planet = 'Mars'
car = "Honda"
```

A common function used on a string is len, which returns the length of the string. We could use this in place of *enumerate* to enumerate through the string.


```python
fruit = 'pineapple'
print(len(fruit)) # 9
```

Because strings are collections, you can obtain substrings using slices. The first number is the starting index, and the second number is the ending index (which is excluded from the substring).

```python
fruit = 'pineapple'
print(fruit[0:2]) # pi
```

Testing for characters in a string are supported using the 'in' operator and index function.

```python
if 'a' in 'pineapple':
    print('a is in pineapple') # a is in pineapple
```

Python also has a few functions to convert the string according to the English language.

```python
print('pineapple'.capitalize())
print('pineapple'.upper())
print('pineapple'.lower())

'''
Returns:
Pineapple
PINEAPPLE
pineapple
'''
```

The code snippet below uses the find() function and string slicing to extract 0.8475.

```python
s = 'X-DSPAM-Confidence:0.8475'

# Find the index of the ':' character in str
index = s.find(':')

# Get the characters after the index and print it as a float
print(float(s[index + 1:]))
# Returns
0.8475
```

Consider a problem where we are given a string with numbers and spaces, and we only need to evaluate the numbers. Therefore it would be nice to find a way to remove all emppty empty spaces. The trim() function would be useful but only if the empty spaces occur at the beginning and end of the string. If there are spaces within the string, then one way to remove them is to do it manually:

```python
def remove_spaces(word):
    ret = ''
    for x in word:
        if x != ' ':
          ret = ret + x
    return ret

if __name__ == '__main__':
    print(remove_spaces('1 2 3 4 5 6'))
# 123456
```

We can solve this issue more succinctly by using the replace() function.

```python
print('1 2 3 4 5 6'.replace(' ', ''))
# 123456
```

Also consider the case where we're given a string and want to check whether it's also a valid number or not. Rather than iterating through the elements of the string and checking whether it's a digit, we could call isdigit() on the entire string.

```python
a='798372'
if a.isdigit() == True:
    print("Yes it's a digit")
```

{% hint style="info" %}
The plain int type is unbounded, so there is no maximum value it can hold.
{% endhint %}

## Code blocks

Python uses indentation to indicate code blocks. When you write a function, code within the block needs to be indented.

```python
def hello():
    print('hello world')

hello()
print('I am not in the function')
```

## Functions

To write a function, use the def statement. Lines that follow which are indented will be part of the function.

```python
def message():
    print('It is raining today')

```

Main functions are executed by comparing the value of __name__.

```python
def main():
    print('hello')

if __name__ == '__main__':
    main()
```

Function arguments appear in the parentheses after the function name. As with declaring variables in Python, you do not specify the variable type.

```python
def print_twice(name):
    print(name * 2)

print_twice('hello ') # Prints hello hello
```

Python has a few useful builtin functions, as the example below shows.

```python
int('32') # Convert a string to an integer
float(32) # Convert an integer to a float
str('32') # Convert to a string
len('Hello World') # Returns the length of a string
```

Python supports functions with a variable number of arguments. In the first case, the variable is prefixed with a single star. It is used as a list.

```python
def line(*words):
	for word in words:
		print(word, end = ' ')
	print()

line('I', 'am', 'a', 'line')
```

The other syntax for variable arguments works like a dictionary.

```python
def line(**words):
	for key, value in words.items():
		print('{0} -> {1}'.format(key, value), end = ' ')
	print()

line(name = 'John', age = 23)
```

## Using modules
Any .py file is a module. Python comes with over 200 modules. To use modules, use the import statement along with the module you want to use. For example:

```python
import math

math.sqrt(4) # Returns 2.0
math.pi # Returns the value of pi

radius = 5
area_of_circle = math.pi * (radius ** 2)
print(area_of_circle)
```

Below is an example of using the random module.

```python
import random

for _ in range(10):
    print(random.randint(0, 10))
```

The example above uses the builtin *range() function. If given a single parameter, it will iterate between 0 and that number exclusive.

```python
for x in range(6):
  print(x)
```

## Handling user input

The builtin input() function can be used to read in input from the keyboard. By default, the value is a string, so if integers or floats are needed, then the value must be converted. This may require the use of exception handling.

```python
import sys

try:
    hours = int(input('Enter Hours: '))
    pay_rate = int(input('Enter rate: '))
except:
    print('Please ener numeric input')
    sys.exit()

# Worked less than 40 hours
if hours <= 40:
    pay = pay_rate * hours
else:
    # Over 40 hours
    overtime_hours = hours - 40
    pay = pay_rate * 40
    overtime_pay = 1.5 * pay_rate * overtime_hours
    pay += overtime_pay

print('Pay: {0}'.format(pay))
```

## Nested and chained conditionals

Unlike Java, nested conditions make use of 'and' and 'or' statements.

```python
import sys

try:
    score = float(input('Enter score: '))
except ValueError as v:
    print('Bad score (not a valid number)')
    sys.exit()

if score >= 0.0 and score <= 1.0:
    if score >= 0.9:
        grade = 'A'
    elif score >= 0.8:
        grade = 'B'
    elif score >= 0.7:
        grade = 'C'
    elif score >= 0.6:
        grade = 'D'
    else:
        grade = 'F'
else:
    print('Bad score')
    sys.exit()

print(grade)
```

## Ternary expressions

In Java, ternary expressions involve the use of *?* and *:* to delineate expressions to evaluate. For example:

```java
String result = statusCode > 0 ? "PASSED" : "FAILED"
```

In Python, the same is acheived with the following blueprint:

```python
condition_if_true if condition else condition_if_false
```

Example:

```python
result = "PASSED" if statusCode > 0 else "FAILED"
```

## Iterations

Python has while and for loops similar to Java.

```python
# basic for loop
for i in range(10):
    print(i)

# while loop. Notice that we don't have a -- operator.
i = 10
while i > 0:
    print(i)
    i -= 1

# for loop used to iterate through an array containing strings
fruits = ['pen', 'pineapple', 'apple', 'pen']
for fruit in fruits:
    print(fruit)
```

To shortcut a for loop, use the *break* keyword.

```python
fruits = ["apple", "banana", "cherry"]
for x in fruits:
  if x == "banana":
      break
  print(x)
```

However at times we need the index of the array. In this case, use the enumerate() function.

```python
fruits = ['pen', 'pineapple', 'apple', 'pen']
for index, value in enumerate(fruits):
    if value == 'apple':
        fruits[index] = 'orange'

print(fruits)
```

## Exceptions

Python has exception handling similar to Java. It is useful, for example, to print a message explaining why the program cannot continue due to an erroneous situation.

The program below tries to open a file, and prints an exception when the file can't be opened.

```python
try:
    f = open('mbox.bin')
    lines = f.readlines()
    f.close()
except:
    print('Unable to open file (the file may not exist')
```

Specific exceptions can be caught. The example below remains in a loop until a number is entered.

```python
while True:
    try:
        x = int(input("Please enter a number: "))
        break
    except ValueError:
        print("Oops!  That was no valid number.  Try again...")
```

Exceptions can have more detailed information, which can be captured in a variable.

```python
import sys

try:
    f = open('myfile.txt')
    s = f.readline()
    i = int(s.strip())
except OSError as err:
    print("OS error: {0}".format(err))
except ValueError:
    print("Could not convert data to an integer.")
except:
    print("Unexpected error:", sys.exc_info()[0])
    raise
```

User-defined exception involve creating a class extending from Exception or another Error class. The example below does this, as well as implementing a finally clause which will be called afterwards.

```python
class NotANumberError(Exception):
    pass

while True:
    try:
        x = int(input("Please enter a number: "))
        break
    except:
        raise NotANumberError('Not a number')
    finally:
        print('Calling cleanup')
```