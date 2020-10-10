# Part 6: Odds and Ends

## Explicit line continuation

If you define a variable and assign a value to it that spans multiple lines, the easy solution is to use the \ character.

```python
s = 'a' + 'b' \
    + 'c' + 'd' \
    + 'e' + 'f'
```

## Implicit line continuation

An alternative is to use ().

```python
s = ('a' + 'b'
    + 'c' + 'd'
    + 'e' + 'f'
)
```

## The type of a function call

We know that if we print the type of a function, that type is function. However, the type of a function return call when that function doesn't return anything is NoneType.

```python
def f(): pass
print(type(f())) # prints NoneType
```

## Complex numbers

Complex numbers are declared using J

```python
f = 6j
```

## Enviroment Variables

Environment variables are accessed using the os module. It is often used to inject values into code without hardcoding it.

```python
import os

ENCRYPTED = os.environ['DB_PASSWORD']

if __name__ == '__main__':
    print(ENCRYPTED)
```

## Interactively loading and running functions

If you have a library and want to invoke a function, you can load the file using the -i option and specifying the function you want to invoke.

```
python -i main.py
f()
```

## Use pdb

Use pdb to debug through your program.

```python
import pdb
pdb.set_trace()
...
```

## Virtual environments

Use virtual environments to install libraries needed by your application and yet isolate it from your system libraries. A popular choice is virtualenv.

```python
virtualenv venv
. ./venv/bin/activate
./venv/bin/pip install <libary>

```