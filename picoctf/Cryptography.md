# C3
First let's see the files we've got at our hands...
```py
achu@air ~ % cat /Users/achu/Downloads/ciphertext <(echo "\n\n") /Users/achu/Downloads/convert.py
DLSeGAGDgBNJDQJDCFSFnRBIDjgHoDFCFtHDgJpiHtGDmMAQFnRBJKkBAsTMrsPSDDnEFCFtIbEDtDCIbFCFtHTJDKerFldbFObFCFtLBFkBAAAPFnRBJGEkerFlcPgKkImHnIlATJDKbTbFOkdNnsgbnJRMFnRBNAFkBAAAbrcbTKAkOgFpOgFpOpkBAAAAAAAiClFGIPFnRBaKliCgClFGtIBAAAAAAAOgGEkImHnIl


import sys
chars = ""
from fileinput import input
for line in input():
  chars += line

lookup1 = "\n \"#()*+/1:=[]abcdefghijklmnopqrstuvwxyz"
lookup2 = "ABCDEFGHIJKLMNOPQRSTabcdefghijklmnopqrst"

out = ""

prev = 0
for char in chars:
  cur = lookup1.index(char)
  out += lookup2[(cur - prev) % 40]
  prev = cur

sys.stdout.write(out)
```
Interesting... We'll start off by reversing the the "encryption" made by replacements in the for-loop by literally just doing it in reverse
```py
>>> with open("/Users/achu/Downloads/ciphertext", "r") as f:
...     c = f.read()
...     
>>> c
'DLSeGAGDgBNJDQJDCFSFnRBIDjgHoDFCFtHDgJpiHtGDmMAQFnRBJKkBAsTMrsPSDDnEFCFtIbEDtDCIbFCFtHTJDKerFldbFObFCFtLBFkBAAAPFnRBJGEkerFlcPgKkImHnIlATJDKbTbFOkdNnsgbnJRMFnRBNAFkBAAAbrcbTKAkOgFpOgFpOpkBAAAAAAAiClFGIPFnRBaKliCgClFGtIBAAAAAAAOgGEkImHnIl'
>>> lookup1 = "\n \"#()*+/1:=[]abcdefghijklmnopqrstuvwxyz"
>>> lookup2 = "ABCDEFGHIJKLMNOPQRSTabcdefghijklmnopqrst"
>>> for v in c:
...     value = lookup2.index(v)
...     cur = (value + prev) % 40
...     decrypted_char = lookup1[cur]
...     decrypted += decrypted_char
...     prev = cur
...     
>>> decrypted
'#asciiorder\n#fortychars\n#selfinput\n#pythontwo\n\nchars = ""\nfrom fileinput import input\nfor line in input():\n    chars += line\nb = 1 / 1\n\nfor i in range(len(chars)):\n    if i == b * b * b:\n        print chars[i] #prints\n        b += 1 / 1\n'
>>> print(decrypted)
#asciiorder
#fortychars
#selfinput
#pythontwo

chars = ""
from fileinput import input
for line in input():
    chars += line
b = 1 / 1

for i in range(len(chars)):
    if i == b * b * b:
        print chars[i] #prints
        b += 1 / 1
```
We get a new python code as our decrypted message
I tried giving original cipher text as input to this new program but that was to no avail.
Let's convert this to python3 before any more shenanigans and try giving the python file itself as the input (because of the comment `#selfinput`)
```py
chars = ""
with open(__file__) as f:
    inp = f.read() # Wasn't too fond of overwriting the builtin input function :p
for line in inp:
    chars += line
b = 1 / 1

for i in range(len(chars)):
    if i == b * b * b:
        print(chars[i]) #prints <- just use the function print, everything else is compatible
        b += 1 / 1
```
If we apply a little bit of brain juice we can see that wasn't the brightest idea because now we reading off of our own edit, instead just give the ORIGINAL file as is to the edited python program
```py
chars = """\
#asciiorder
#fortychars
#selfinput
#pythontwo

chars = ""
from fileinput import input
for line in input():
    chars += line
b = 1 / 1

for i in range(len(chars)):
    if i == b * b * b:
        print chars[i] #prints
        b += 1 / 1
"""

b = 1 / 1
for i in range(len(chars)):
    if i == b * b * b:
        print(chars[i]) #prints <- just use the function print, everything else is compatible
        b += 1 / 1
```
This produces the output
```
a
d
l
i
b
s
```
Wrap that around a flag boilerplate and we get `picoCTF{adlibs}`, and that's our flag!


# Custom encryption
TBD


# miniRSA
TBD


