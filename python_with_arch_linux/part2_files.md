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

Python has a convenient function readlines() to read the contents of a file into a list. The code below finds all the email addresses in mbox.txt, discarding duplicates. The two most common functions used to accomplish this are startswith() and find(). Note that these are functions for strings, not files.

```python
with open('mbox.txt') as f:
    lines = f.readlines()

email_address_list = []

for line in lines:
    if line.lower().startswith('from '):
        interesting_line = line.lower().split('from ')[1]
        email_address = interesting_line.split()[0]
        if email_address.find('@') == -1:
            continue
        if email_address not in email_address_list:
            email_address_list.append(email_address)
    
email_address_list.sort()

for email_address in email_address_list:
    print(email_address)
```
