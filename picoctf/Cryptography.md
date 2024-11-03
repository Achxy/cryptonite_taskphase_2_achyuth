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

!TODO: Finish this...


# Custom encryption
TBD


# miniRSA
TBD


