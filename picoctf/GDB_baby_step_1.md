We have to find the content of the `eax` register and convert that hex value to decimal value and wrap it around the `picoCTF{...}` flag boiler.
Intuitively, the given file has to bin executable.
```bash
achu@air ~ % file /Users/achu/Downloads/debugger0_a
/Users/achu/Downloads/debugger0_a (1): ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=15a10290db2cd2ec0c123cf80b88ed7d7f5cf9ff, for GNU/Linux 3.2.0, not stripped
achu@air ~ % 
```
Well... that confirms it.

We'll get the disassemble using `objdump` with flag `-d`, and just use `grep` to find the `eax` call
```bash
achu@air ~ % objdump -d /Users/achu/Downloads/debugger0_a | grep eax
    1138: b8 42 63 08 00               	movl	$549698, %eax           # imm = 0x86342
achu@air ~ % echo $((16#86342))  
549698
achu@air ~ % 
```
There's only one `eax`, and we'll convert `0x86342` to decimal base to get `549698` and we'll wrap it around the flag format to form `picoCTF{549698}`.
