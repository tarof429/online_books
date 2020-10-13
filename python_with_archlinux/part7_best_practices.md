# Part 7: Best Practices

This section is based on the content from https://effectivepython.com/. 

## Use `f strings` over other methods of string interpolation.

```python
world = 'world'
print(f'hello {world}')
# Output: hello world
```

## Prefer multiple assignment unpacking over indexing

Consider the case where we need to iterate over a map. One way to do this is with a *for...in* loop.

```python
ec2s = {
    'id-5': 'Ubuntu',
    'id-3': 'Amazon Linux 2',
    'id-1': 'RedHat Linux'
}

for ec2 in ec2s.items():
    print(ec2)
# ('id-5', 'Ubuntu')
# ('id-3', 'Amazon Linux 2')
# ('id-1', 'RedHat Linux')
```

And what if we wanted just the IDs? One way to do this is to iterate through the dictionary and each each ID to a list. The *tuple* function used here unpacks each item.

```python
ec2sTuple = tuple(ec2s.items())
ids = []
for item in ec2sTuple:
    ids.append(item[0])

print(ids)
```

A more *pythonic* approach is to use *map*, because we want to perform the same operation on each element.

```python
def get_id(x):
    return x[0]

ids = list(map(get_id, ec2sTuple))
print(ids)
# ['id-5', 'id-3', 'id-1']
```

Instead of passing in a function, we could just use a *lambda* expression. This uses far less code and is more readable as a result.

```python
ids = list(map(lambda x: x[0], ec2sTuple))
print(ids)
```

And finally we come to the most *pythonic* expression to iterate through a list: *list comprehension*. The challenge here in my opinion is considering list comprehension as a type of unpacking.

```python
ids = [x[0] for x in ec2sTuple]
print(ids)
``` 