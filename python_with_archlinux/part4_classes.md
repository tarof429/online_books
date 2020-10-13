# Part 4: Classes

Classes are used to encapsulate data and its operations. This allows programmers to focus on the larger program logic instead of the details needed to manipulate data.

Below is an example of creating and using a class. Note that unlike Java, there is no keyword like *new* that is needed to create new instances of the class.

```python
class Animal:
		
	def bark(self):
		print('Bark!')

dog = Animal()
dog.bark()
```

The class above does not define a constructor so an implicit one is used. Python doesn't have the ability to define multiple constructors like in Java. Instead, we can define a constructor with default values in case none is provided.

```python

class Animal:

	def __init__(self, name='Dog'):
		self.name = name
	
	def bark(self):
		print('I am a {0}. Bark!'.format(self.name))

dog = Animal()
dog.bark()

cat = Animal('Cat')
cat.bark()
```

Python classes also suppot inheritance. To do this, after the class declaration enclose the superclass in parenthesis.

```python
class Animal:
	def walk(self):
		print('I like to walk')

class Dog(Animal):
	def __init__(self, name='Dog'):
		self.name = name

	def bark(self):
		print('Woof! I am a {0}'.format(self.name))
	
class Cat(Animal):
	def __init__(self, name='Cat'):
		self.name = name

	def meow(self):
		print('Meow! I am a {0}'.format(self.name))

dog = Dog()
dog.bark()
dog.walk()

cat = Cat()
cat.meow()
cat.walk()
```

In case we want to define a class without any methods, use the *pass* keyword. The *pass* keyword can also be used to declare methods without an implementation.

```python
class Animal:
	pass
```
