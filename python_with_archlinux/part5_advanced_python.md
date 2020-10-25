# Part 5: Advanced Python

## Doctest

Doctest allows us to run tests on a function based on docstrings. A simple example is shown below.

```python
"""
This function returns the sum of two numbers.

>>> add_two_numbers(3, 5)
8
"""
def add_two_numbers(x, y):
    return x + y 

if __name__ == '__main__':
    import doctest
    doctest.testmod()
```

When run, if there are no errors there is no output. To display verbose output, run the following.

```python
python -m doctest -v main.py
```

Docstrings can occur within the function as well.

```python
#!/usr/bin/python

# import testmod for testing our function 
from doctest import testmod 

# define a function to test 
def factorial(n): 
    ''' 
    This function calculates recursively and 
	returns the factorial of a positive number. 
	Define input and expected output: 
	>>> factorial(3) 
	6
	>>> factorial(5) 
	120
	'''
    if n <= 1: 
        return 1
    return n * factorial(n - 1) 

# call the testmod function 
if __name__ == '__main__': 
    testmod(name ='factorial', verbose = True)
```

## Generators

Generators are functions that carry state, so that repeated calls to the function can yield different results. 

The example below generates a key similar to docker IDs.

```python
import random

# Generate a value for a key. Subsequent calls will return a new value for the
# key. If dups is set to False, then subsequent calls will return a unique
# value.
def key_generator(size, dups=True):
    alpha_list = ['a', 'b', 'c', 'd', 'e', 'f']
    num_list = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

    key_value_list = alpha_list + num_list

    # If the key size is greater than the length of the key value list, 
    # and we request that there be no duplicate values in the key, then
    # raise an exception
    if dups == False and size > len(key_value_list):
        raise Exception('Not allowed')

    for i in range(size):
        index = random.randint(0, len(key_value_list) - 1)
        if not dups:
            yield key_value_list.pop(index)
        else:
            yield(key_value_list[index])

key = ''

for item in key_generator(16, True):
    key += str(item)

print(key)
```

## Iterators

To implement your own iterator, a class must implement __iter__() and __next__(). Below is an example to return a list of prime numbers.

```python
class Prime:

	def __init__(self, max = 0):
		self.primes = [2, 3, 5, 7, 11, 13, 17, 19]

		if max > len(self.primes):
			raise Exception('Over the limit')
		self.max = max
	
	# Return the iterator object itself
	def __iter__(self):
		self.n = 0
		return self
		
	# Return the next item in the sequence
	def __next__(self):
		if self.n <= self.max:
			result = self.primes[self.n]
			self.n += 1
			return result
		else:
			raise StopIteration

for p in Prime(6):
	print(p)
```

## Functional programming

In Python, functions are first class citizens and they can be assigned to variables and invoked as if we are calling the actual function.

```python
def hello():
	print('hello')

f = hello
f() # prints hello
```

One use case is to determine which function to invoke based on an environment variable.

```python
def build_mock():
	print('Doing a mock build')

def actual_build():
	print('Doing the actual build')

BUILD_ENV = 'mock'

build = build_mock if BUILD_ENV == 'mock' else actual_build

build()
```

Another feature of functional programming is that functions can passed to functions as arguments. 

```python
def build_mock():
	print('Doing a mock build')

def actual_build():
	print('Doing the actual build')

def build(func):
	func()

BUILD_ENV = 'mock'

build(build_mock) if BUILD_ENV == 'mock' else build(actual_build)
```

And yet another possibility is to return functions from functions.

```python
def build(BUILD_ENV):
	if BUILD_ENV == 'mock':
		def func():
			print('Doing a mock build')
	else:
		def func():
			print('Doing the actual build')

	return func

f = build('mock')
f()
```

Functions invoked in this way can also have return values, even tuples.

```python
def build(BUILD_ENV):
	if BUILD_ENV == 'mock':
		def func():
			return ('PASS', 'Doing a mock build')
	else:
		def func():
			return ('FAIL', 'Doing the actual build')

	return func

func = build('mock')

status, message = func()

print(f'status: {status} message: {message}')
```

When we define a function that returns another function, the function we return still has access to the internal scope of the function that returned it. This is called closure. The function above has been modified so that the inner function func() accesses the user variable passed in from outside.

```python
def build(BUILD_ENV, user):
	if BUILD_ENV == 'mock':
		def func():
			return ('PASS', 'Doing a mock build', user)
	else:
		def func():
			return ('FAIL', 'Doing the actual build', user)

	return func

func = build('mock', 'root')

status, message, user = func()

# Returns user: root status: PASS message: Doing a mock build
print(f'user: {user} status: {status} message: {message}')
```

Python has excellent support for applying functional programming principles on lists. Below is a list of Python functions which can be applied to lists.

- map
- filter 
- reduce
- list comprehensions

## Map

A map changes each item in a list to another value. For example, this allows us to convert a list of miles to kilometers.

The syntax of map in Python is:

```python
map(func, *iterables)
```

This is illustrated in the example below.

```python
miles = [1, 120, 445]

def miles_to_kilometers(x):
    return x * 1.60934

# Convert a list of miles to kilometers
kilometers = list(map(miles_to_kilometers, miles))

# Prints [1.60934, 193.1208, 716.1563]
print(kilometers)
```

## Filter

The filter function is used to return a subset of a list based on criteria defined in the form of a function. The syntax of filter is similar to map.

```python
filter(func, *iterables)
```

The example below creates a list of values only if they are greater than 50.

```python
miles = [1, 30, 90, 120, 445]

def miles_over_50(x):
	return x > 50

l = list(filter(miles_over_50, miles))

# Returns [90, 120, 445]
print(l)
```

We can combine the two examples above to get a list of temperatures only if their value in miles was over 50.

```python
miles = [1, 30, 90, 120, 445]

def miles_over_50(x):
	return x > 50

def miles_to_kilometers(x):
	return x * 1.60934

sublist = list(filter(miles_over_50, miles))

kilometers = list(map(miles_to_kilometers, sublist))

# Returns [144.8406, 193.1208, 716.1563]
print(kilometers)
```

A map can be rewritten using lamda expressions which defines the input parameter and what we want to do with it. The syntax of lambda is:

```python
lambda parameter_list: expression
```

A snippet is shown below.

```python
kilometers = list(lambda x: x * 1.60934, sublist)
```

lambdas can only be one line long, so although we can't use if/else statements, we can use ternary operators.

```python
miles = [1, 30, 90, 120, 445]

sublist = list(filter(lambda x: True if x > 50 else False, miles))
kilometers = list(map(lambda x: x *  1.60934, sublist))

# Returns [144.8406, 193.1208, 716.1563]
print(kilometers)
```

## zip function

This function creates a list of tuples comprised of elements from multiple lists.

```python

distance = 0

for x, y in zip(strand_a, strand_b):
    if x != y: distance += 1
```

## List comprehensions

A list comprehension is a Python-native way to perform map, filter and lambdas. The general syntax is:

```python
[ expression for item in list if conditional ]
```

For example:

```python
miles = [1, 30, 90, 120, 445]

# Filtering with list comprehensions
sublist = [x for x in miles if x > 50]

# Map with list comprehensions
kilometers = [x *  1.60934 for x in sublist]

# Returns [144.8406, 193.1208, 716.1563]
print(kilometers)
```

Below we create a one dimensional array.

```python
C = [ i  for i in range(10)]
```

A typical problem involving list comprehension is how to convert an iterative approach to a functional one. For example, the code below finds the count of all directories.

```python
import os, os.path

path='.'

dirs = []

with os.scandir(path) as it: 
    for entry in it: 
        if entry.is_dir():
            dirs.append(entry.name)

print('{} projects'.format(len(dirs)))
```

Now instead of using os.scandir(), we can use os.listdir() so we immediately get a list back instead of an iterator.

```python
import os, os.path

dirs = [d for d in os.listdir('.') if os.path.isdir(d)]

print('{} projects'.format(len(dirs)))
```                    

## Reduce

Reduce is a type of function which reduces a sequence into a single value. This can be used, for example, to compute sums. To use reduce, we need to define a function. The syntax of the function is a little unusual in that the first argument is an accumulator.

```python
def func(acc, element):
    # etc.
```

When we use the reduce() function, the arguemnts are the name of the function and a list as we've seen before with map and filter. There is additionally a third argument, the initial value for the accumulator.

```python
reduce(func, list, initial_value)
```

Using reduce, we can get the sum of the list.

```python
from functools import reduce

miles = [1, 30, 90, 120, 445]

# Filtering with list comprehensions
sublist = [x for x in miles if x > 50]

# Map with list comprehensions
kilometers = [x *  1.60934 for x in sublist]

def get_sum(acc, element):
    return acc + element

# Get the sum
sum = reduce(get_sum, kilometers)

# Prints 1054.1177
print(sum)
```

Using reduce with list comprehension.

```python
from functools import reduce

fruits = [
    {'name': 'apple', 'price': 20},
    {'name': 'pear', 'price': 30},
    {'name': 'banana', 'price': 40}
]

def get_sum(acc, element):
    return acc + element


prices = [fruit['price'] for fruit in fruits]
sum = reduce(get_sum, prices)
print(sum)
```

## System Calls

There are a few ways to run shell commands in Python.

The first is to use the system class in the os module. 

```python
import os

os.system('ls /')
```

One big limitation is that the output is sent to the console and you cannot store it in a variable. An alternative is to use methods in the subprocess module.

```python
import subprocess
output = subprocess.call("ls")
print(output)
```

One problem with subprocess is that it doesn't handle command arguments in the way you expect. For example, the code below will result in an exception.

```python
import subprocess
output = subprocess.call("ls -l /")
print(output)
# Results in FileNotFoundError: [Errno 2] No such file or directory: 'ls /'
```

One solution is to use getoutput() and getstatusoutput() methods instead.

```python
import subprocess

stdout = subprocess.getoutput('ls -l /')
print(stdout)

exitcode, stdout = subprocess.getstatusoutput('df -h')
print(exitcode)
print(stdout)
```

## JSON

The json module provides methods to parse JSON.

```python
weapon = '{"name": "dagger", "damage": "1-6", "type": "melee", "minimum-level": "1", "cost": "3", "minimum-strength": "6", "minimum-dexterity": "3"}'

weapon_json = json.loads(weapon)

print(weapon_json['name']) 
```

Now if we are given a Python dictionary instead of a string, we will need to convert it to a string first.

```python
import json

weapon = {"name": "dagger", "damage": "1-6", "type": "melee", "minimum-level": "1", "cost": "3", "minimum-strength": "6", "minimum-dexterity": "3"}

weapon_str = json.dumps(weapon)
print(type(weapon_str)) # string
weapon_json = json.loads(weapon_str)
print(weapon_json['name']) 
```

If the json data was in a file, we can pass in the file pointer to the loads() function.

```python

```