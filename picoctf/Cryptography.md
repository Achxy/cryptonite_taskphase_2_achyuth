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
Textbook RSA problem where the exp is small, also it's given that the word `pico` is contained within the flag
> pico is in the flag, but not at the beginning

so, we'll brute guesses and check if `pico` is a substring of produced byte string.
```py
from itertools import count

from Crypto.Util.number import long_to_bytes

n, c, e = (
    1615765684321463054078226051959887884233678317734892901740763321135213636796075462401950274602405095138589898087428337758445013281488966866073355710771864671726991918706558071231266976427184673800225254531695928541272546385146495736420261815693810544589811104967829354461491178200126099661909654163542661541699404839644035177445092988952614918424317082380174383819025585076206641993479326576180793544321194357018916215113009742654408597083724508169216182008449693917227497813165444372201517541788989925461711067825681947947471001390843774746442699739386923285801022685451221261010798837646928092277556198145662924691803032880040492762442561497760689933601781401617086600593482127465655390841361154025890679757514060456103104199255917164678161972735858939464790960448345988941481499050248673128656508055285037090026439683847266536283160142071643015434813473463469733112182328678706702116054036618277506997666534567846763938692335069955755244438415377933440029498378955355877502743215305768814857864433151287,
    1220012318588871886132524757898884422174534558055593713309088304910273991073554732659977133980685370899257850121970812405700793710546674062154237544840177616746805668666317481140872605653768484867292138139949076102907399831998827567645230986345455915692863094364797526497302082734955903755050638155202890599808146956044568639690002921620304969196755223769438221859424275683828638207433071955615349052424040706261639770492033970498727183446507482899334169592311953247661557664109356372049286283480939368007035616954029177541731719684026988849403756133033533171081378815289443019437298879607294287249591634702823432448559878065453908423094452047188125358790554039587941488937855941604809869090304206028751113018999782990033393577325766685647733181521675994939066814158759362046052998582186178682593597175186539419118605277037256659707217066953121398700583644564201414551200278389319378027058801216150663695102005048597466358061508725332471930736629781191567057009302022382219283560795941554288119544255055962,
    3,
)


def invpow(x, n):
    if x < 2:
        return x
    low, high = 1, x
    while low <= high:
        mid = (low + high) // 2
        mid_pow = mid**n
        if mid_pow == x:
            return mid
        elif mid_pow < x:
            low = mid + 1
        else:
            high = mid - 1
    return high


for i in count():
    flag = long_to_bytes(invpow(i * n + c, e))
    if b"pico" in flag:
        print(flag.decode().strip())
        break
```
And we get the flag: `picoCTF{e_sh0u1d_b3_lArg3r_6e2e6bda}`
(`NOTE`: This does take a while to run, we could potentially speed this up by not going for a pure python approach and implement it using C with exposure to FFI or just use existing tools like `gmpy2`)
