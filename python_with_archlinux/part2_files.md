# Part 2: Files

Opening files for reading is performed using the built-in function open(). Remember to close the file handle afterwards.

```python
f = open('mbox.txt')
for line in f:
    print(line)
f.close()
```

Alternatively, the *with* statement automatically closes the file.

```python
with open('mbox.txt') as f:
    for line in f:
        print(line)
```

To read the entire contents of the file and store it in a variable, use the read() method. The example below reads in the contents of mbox.txt and copies it to another file, output.txt. We must specify the mode when writing.

```python
with open('mbox.txt') as f:
    lines = f.read()

with open('output.txt', 'w') as f:
    f.write(lines)
```

Python has a convenient function readlines() to read the contents of a text file into a list. The code below finds all the email addresses in mbox.txt, discarding duplicates. The two most common functions used to accomplish this are startswith() and find(). Note that these are functions for strings, not files.

```python
with open('mbox.txt') as f:
    lines = f.readlines()
```

## Excercise: Printing contents of a file

### Problem

Suppose we have a file called `pi.txt`. 

{% code title="pi.txt" %}
```
 3.14159
   265358979
   32384626433
```
{% endcode }

Write a program to print the contents of the file exactly.

### Solution

```python
with open('pi.txt') as f:
    for line in f:
        print(line.rstrip())
```

The reason why we need to use `rstrip()` is that print adds its own newline, so we should remove the newline from `line` to match the file content exactly.

Alternatively we could have read the lines into a list.

```python
with open('pi.txt') as f:
    lines = f.readlines()

for line in lines:
    print(line.strip())
```

## Excercise: Listing filesystems

### Problem

Given the content of /etc/fstab to be:

```
# Static information about the filesystems.
# See fstab(5) for details.

# <file system> <dir> <type> <options> <dump> <pass>
# /dev/nvme0n1p3
UUID=6a572148-dba3-484a-87e6-ecbb7c0c9eac	/         	ext4      	rw,relatime	0 1

# /dev/nvme0n1p2
UUID=85e1163f-b416-474e-bc90-e1ad4557c8eb	/boot     	ext4      	rw,relatime	0 2

# /dev/nvme0n1p1
UUID=fd586818-a832-411e-a22b-131e62b84fa2	none      	swap      	defaults,pri=-2	0 0

# /dev/sda1
UUID=39cbbf4c-4a19-4a1c-9842-af10d642535b	/home		ext4		defaults	0 2

# /dev/sdb1
UUID=ac74d400-dd7d-45e2-b19f-55eb034f3b33 	/mnt/Backup	ext4		nofail,x-systemd.device-timeout=10   0  2
```

List the filesystems.

### Solution

Below is a solution that uses list comprehension. This is preferred over using multiple for loops. Because we have multiple characters that we want to use in our conditional statement, we use a tuple.

```python
with open('fstab.txt') as fstab:
    fstablines = fstab.readlines()

# Use list comprehension over for loops
# Slightly longer approach, but not bad
# fstabGoodLines = [x for x in fstablines if not x.startswith('#')]
# fstabGoodLines = [x for x in fstabGoodLines if not x.startswith('\n')]
# fstabGoodLines = [x.strip() for x in fstabGoodLines]
# fstabGoodLines = [x.split()[1] for x in fstabGoodLines]

# A one-liner, although more difficult
fstabGoodLines = [x.strip().split()[1] for x in fstablines if not x.startswith(('#', '\n'))]

for x in fstabGoodLines:
    print('filesystem: ' + x)

# Output
# $ python main.py 
# filesystem: /
# filesystem: /boot
#filesystem: none
# filesystem: /home
# filesystem: /mnt/Backup

```