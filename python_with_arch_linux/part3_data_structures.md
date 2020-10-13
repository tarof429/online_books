# Part 3: Data Structures

## Lists

To create a list, enclose a set of items in [].

```python
fruits = ['apple', 'orange', 'banana']

for fruit in fruits:
    print(fruit)
```

Just as with other programming languages, elements in the list can be accessed and modified using indexes.

```python
print(fruits[0])
fruits[0] = 'kiwi'
print(fruits[0])
```

Python also supports negative indexes and ranges.

```python
print(fruits[-1]) # Print the last element
print[fruits[0:2]] # Print the first element up to but not including the element with index 2, i.e.: apple and orange
```

To remove elements in a list, you can use the del operator and provide an index. This operator modifies the original list.

```python
mylist = ['a', 'b', 'c', 'd']
del mylist[0]
del mylist[-1]
```
If you want to remove an element in the list and have it returned, then use the pop function. 

```python
mylist = ['a', 'b', 'c', 'd']
first_element = mylist.pop(0)
print(first_element)
```

There are several ways to append items to a list. 

```python

a = ['a', 'b', 'c']
a += ['d', 'e']
# abcde
print(a)

# abcdefg
a += 'fg'
print(a)

# abcdefghi
a.extend('hi')
print(a)
```

Python can have nested lists. The code below prints the character 'z' in 'baz'.

```python
# prints 'z'
x = [10, [3.141, 20, [30, 'baz', 2.718]], 'foo']
print(x[1][2][1][2])
```

Using the same list, we are able to print a slice or range of elements.

```python
# prints 'baz', 2.718
x = [10, [3.141, 20, [30, 'baz', 2.718]], 'foo']
print(x[1][2][1:])
```

Below is a function to read a file and get the unique words in it.

```python
def get_unique_words_from_file(filename):
    unique_list = []

    with open(filename) as f:
        for line in f:
            words = line.split()
            for word in words:
                if word not in unique_list:
                    unique_list.append(word)

    unique_list.sort()
    print(unique_list)

get_unique_words_from_file('romeo.txt')
```

There is a simple way to check if something is in a list.

```python
l = ['a', 'b', 'c']

if 'd' not in l:
    l.append('d')
```

The example below shows how to use a list to check if a word is an isogram. The first solution uses two if statements to skip over characters we don't want to evaulate.

```python
def is_isogram(string):
    l = []

    for c in string.lower():
        if c == '-': continue
        elif c == ' ': continue
        elif c not in l:
            l.append(c)
        else:
            return False

    return True
```

The second solution combines the first two if statement by making use of a list and the in keyword.

```python
def is_isogram(string):
    l = []

    for c in string.lower():
        # Ignore '-' and whitepace characters
        if c in ('-', ' '): continue

        # Check if the character was not encountered before.
        # If encountered previously, then string is not an isogram.
        # Otherwise, add the character to our list.
        elif c in l:
            return False
        else:
            l.append(c)

    # Since we have gone through all the characters in string and have
    # not found it to be duplicated, it must be an isogram.
    return True 
```

## Slices

If you want a sublist, you can use a slice. The second element in the slice is NOT included in the result.

```python
# Prints 1, 2
a = [1, 2, 7, 8]
print(a[0:2])

```

The following uses a negative value for the second element in the slice. This means that we do NOT include the last element but the one after if (from the back of the list).

```python
# Prints ['b', 'c']. The way it works is that first we find mylist[1] which is 'b'. Then we go to the end of the index and go back on, or 'c'. We do not go relative to the first index.
mylist = ['a', 'b', 'c', 'd']
newlist = mylist[1:-1]
print(newlist)
```

We can use slices to insert elements into the list.

```python
# [1, 2, 3, 4, 5, 6, 8]
a = [1, 2, 7, 8]
a[2:3] = [3, 4, 5, 6]
print(list(a))
```

There is a feature called **extended slices**. What you do is specify a third element, called step argument.

```python
# Starting from mylist[1], then choose every element, 2 at a time. We get ['b', 'd']
mylist = ['a', 'b', 'c', 'd']
newlist = mylist[1::2]
print(newlist)
```

This can be used to get every even element

```python
# prints ['a', 'c']
mylist = ['a', 'b', 'c', 'd']
newlist = mylist[::2]
print(newlist)
```

Similarly to get the odd elements.

```python
mylist = ['a', 'b', 'c', 'd']
newlist = mylist[1::2]
print(newlist)
```

And even reverse the list.

```python
# prints ['d', 'c', 'b', 'a']
mylist = ['a', 'b', 'c', 'd']
newlist = mylist[::-1]
print(newlist)
```

Be careful of *empty slices*. This is where a slice does not specify an element. 

```python
# An attempt was made to remove a[2] but it doesn't work.
a = [1, 2, 3, 4, 5]
a[2:2] = []
print(a)
```

Whereas the following removes 3 from the list.

```python
a = [1, 2, 3, 4, 5]
a[2:3] = []
print(a)
```

## Sorting Lists

There are two ways to sort a list. 

A list has a built in function you can use to sort it in place.

```python
scores = [20, 10, 30]
scores.sort()
print(scores
```

There is also a function that returns a sorted list.

```python
scores = [20, 10, 30]
sorted_scores = sorted(scores)
print(sorted_scores)
```

If you want to reverse the sorted order, you can specify the reverse parameter.

```python
scores = [20, 10, 30]
scores.sort(reverse=True)
print(scores)

scores = [20, 10, 30]
sorted_scores = sorted(scores, reverse=True)
print(sorted_scores)
```

There is also a reversed() function that can be called against a list, but in this case, remember that you need to convert the output to a list.

```python
scores = [20, 10, 30]
sorted_scores = list(reversed(sorted(scores)))
print(sorted_scores)
```

## Tuples

An immutable list is called a tuple.

```python
t = ('foo', 'bar', 'baz')

# Error!
t[1] = 'qux'
t(1) = 'qux'
t[1:1] = 'qux'
```

The parenthesis are optional. The important thing here is the comma, not the parenthesis.

```python
t = 'foo', 'bar', 'baz'
print(t[0])
```

To create a tuple with one element, you must include a trailing comma.

```python
t = ('foo',)
print(t[0])

a = 'foo',
print(a[0])
```

As with lists, you can specify slices in tuples.

```python
# Prints 5. We start from 2, then move 3 spaces
a, b, c = (1, 2, 3, 4, 5, 6, 7, 8, 9)[1::3]
print(b)
```

As we've seen with a list, tuples can be reversed using [::-1].

```python
# First the tuple is created to contain -5, 5 (in this order). However, the
# slice syntax [::-1] reverses this, so that when we assign x and y, the values
# are unchanged.
x = 5
y = -5
x, y = (y, x)[::-1]

```

## Stacks

Stacks work on the principle "last in first out", or LIFO. Suppose you have a stack of dirty dishes. A new dish is added to the top of the stack. However to get a dish from the stack, we get it at the bottom. Another way to think of a stack is a line at a supermarket. People make a line at a checkout stand. When the line reaches a certain number of people, a cash registrar takes the person who was at the end of the line. 

The Python list has two methods that allow us to implement a stack. The **append()** method is used to *push* items onto the stack. The **pop()** method is used to *pop* items from the stack. 

```python
stack = ["Java", "C", "Python", "Go"] 
stack.append("Rust")

# Should print Rust
print(stack.pop())
```

## Queues

Stacks on the other hand work on the principle "first in first out", or FIFO. This is equivalent to a checkout stand at a supermaket where the person who was first in line is at the front of the line, and will be the first to pay at the registrar.

```python
queue = ["Java", "C", "Python", "Go"] 
queue.append("Rust")

# Should print Java
print(queue.pop(0))

# Contains the list without Java
print(queue)
```

Python has a special library in the collections module which allows for efficient methods to add and remove items from the front and end of a queue or stack. Below is an implementation of a stack.

```python
from collections import deque

languages = deque()

languages.append("Java")
languages.append("C")
languages.append("Python")
languages.append("Go")


# Should print Java
print(languages.popleft())

# Contains the list without Java
print(languages)
```

So in general, to implement a queue or a stack in python, use the **append()** and **pop()** methods on a list. The dequeue library is optimized for this.

## The any() method

The any() method evaluates to true if an item in a list contains True.

```python
pass_status = [False, False, False, True, False]

# Prints a message because an item in the list was True
if any(pass_status):
    print('A test passed')
```

## Dictionaries

Dictionaries lets you create assocations between keys and values. There are two ways of creating a dictionary. One way is to define the variable as a type with braces.

```python
mydictionary = {}
print(type(mydictionary)) # Prints <class 'dict'>
mydictionary['key'] = 'value'
print(mydictionary['key'])
```

Another way is to use the dict() function.

```python
anotherdict = dict()
print(type(anotherdict))
```

As seen above, to add an item, we simply use square brackets around the key, followed by the value it should represent. This is repeated below.

```python
build['id'] = 1234
build['status'] = 'pass'
```

An alternative way of creating a dictionary is to use colons between key/value pairs, as shown below.

```python
plant = {
    'name': 'oak',
    'age': 50,
    'type': 'tree'
}

print(plant['name']) # Returns oak
```

Let's see how to iterate through the items in a dictionary. Given the code below.

```python
a_dict = {'color': 'blue', 'fruit': 'apple', 'pet': 'dog'}
for key in a_dict:
    print(key)
```

will we see the keys, the values, or both? The answer is the keys only.

```python
# output
color
fruit
pet
```

This is because the iter() method in dictionaries will return the key by default. We can leverage this fact to iterate through the values by making a simple code modification.

```python
for key in a_dict:
    print(a_dict[key])
```

To change a value in a dictionary, specify a key that already exists and set it's value. 

```python
plant['type'] = 'native tree'
```

What happens when we try to get the value for a key that isn't in the dictionary? You will get a KeyError.

```python
print(mydictionary['color']) # Returns KeyError: 'color'
```

The *in* keyword can be used to check if a key exists in a dictionary.

```python
if 'age' in plant and plant['age'] > 50:
	print(plant['age'])
else:
	print('No plant like that exists')
```

A practical use of dictionaries is to provide a set of counters. For example, we can count the number of times a letter occurs in a word. However, there is a problem if we
try to make the code concise and we are dealing with a test for a key that doesn't exist.


```python
# Compiles, but at runtime we will get a KeyError for the first letter
test_word = input('Enter a word: ')
print('Checking {0}...'.format(test_word))

d = dict()

for letter in test_word:
	if letter not in d:
		d[letter] = d[letter] + 1

print(d)
```

One possible way is to set the value to 1 if the letter is not in the dictionary.

```python
for letter in test_word:
	if letter not in d:
		d[letter] = 1
	else:
		d[letter] = d[letter] + 1

print(d)
```

Another way is to use the get() method of a dictionary. This method attempts to get the value of the key; if it doesn't exist, then we can have it default to a value (in this case, 0)

```python
for letter in test_word:
	d[letter] = d.get(letter, 0) + 1

print(d)
```

Dictionaries can be nested.

```python
plants = {
    'plant1' : {
        'name': 'oak',
        'age': 50,
        'type': 'tree'
    },
    'plant2' : {
        'name': 'apple',
        'age': 12,
        'type': 'tree'
    }
}

for plant in plants:
    keys = plant.keys()
    for key in keys:
        print(key)
```

To remove an item from a dictonary, use popitem().

```python
d = {'foo': 1, 'bar': 2, 'baz': 3}
while d:
    print(d.popitem())
print('Done.')

# Result:
# ('baz', 3)
# ('bar', 2)
# ('foo', 1)
# Done.
```

You can also say which element you want to pop.

```python
d.popitem('foo')
```

While popitem() can be used in an ieration, del cannot be used.

```python
prices = {'apple': 0.40, 'orange': 0.35, 'banana': 0.25}
for key in prices.keys():
    if key == 'orange':
        del prices[key]

# Result: RuntimeError: dictionary changed size during iteration
```
