# GDB baby step 1
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


# ARMssembly 1
Let's see what the assembly file does

```s
achu@air ~ % cat /Users/achu/Downloads/chall_1\ \(1\).S 
	.arch armv8-a
	.file	"chall_1.c"
	.text
	.align	2
	.global	func
	.type	func, %function
func:
	sub	sp, sp, #32
	str	w0, [sp, 12]
	mov	w0, 87
	str	w0, [sp, 16]
	mov	w0, 3
	str	w0, [sp, 20]
	mov	w0, 3
	str	w0, [sp, 24]
	ldr	w0, [sp, 20]
	ldr	w1, [sp, 16]
	lsl	w0, w1, w0
	str	w0, [sp, 28]
	ldr	w1, [sp, 28]
	ldr	w0, [sp, 24]
	sdiv	w0, w1, w0
	str	w0, [sp, 28]
	ldr	w1, [sp, 28]
	ldr	w0, [sp, 12]
	sub	w0, w1, w0
	str	w0, [sp, 28]
	ldr	w0, [sp, 28]
	add	sp, sp, 32
	ret
	.size	func, .-func
	.section	.rodata
	.align	3
.LC0:
	.string	"You win!"
	.align	3
.LC1:
	.string	"You Lose :("
	.text
	.align	2
	.global	main
	.type	main, %function
main:
	stp	x29, x30, [sp, -48]!
	add	x29, sp, 0
	str	w0, [x29, 28]
	str	x1, [x29, 16]
	ldr	x0, [x29, 16]
	add	x0, x0, 8
	ldr	x0, [x0]
	bl	atoi
	str	w0, [x29, 44]
	ldr	w0, [x29, 44]
	bl	func
	cmp	w0, 0
	bne	.L4
	adrp	x0, .LC0
	add	x0, x0, :lo12:.LC0
	bl	puts
	b	.L6
.L4:
	adrp	x0, .LC1
	add	x0, x0, :lo12:.LC1
	bl	puts
.L6:
	nop
	ldp	x29, x30, [sp], 48
	ret
	.size	main, .-main
	.ident	"GCC: (Ubuntu/Linaro 7.5.0-3ubuntu1~18.04) 7.5.0"
	.section	.note.GNU-stack,"",@progbits
achu@air ~ % 
```
Let's see what's going on...
```s
ldr w0, [sp,20]      w0 = 3
ldr w1, [sp,16]      w1 = 87
lsl w0, w1, w0       w0 = 87 << 3 = 87 * 8 = 696
str w0, [sp,28]      Store 696 at [sp, 28]
```
then onto the next phase of this
```s
ldr w1, [sp,28]      w1 = 696
ldr w0, [sp,24]      w0 = 3
sdiv w0, w1, w0      w0 = 696 / 3 = 232
str w0, [sp,28]      Store 232 at [sp, 28]
```
finally
```s
ldr w1, [sp,28]      w1 = 232
ldr w0, [sp,12]      w0 = arg (picoctf wants us to find this - arg to be zero, convert that to hex)
sub w0, w1, w0       w0 = 232 - arg
str w0, [sp,28]      Store (232 - arg) at [sp, 28]
```
then return
```s
ldr w0, [sp,28]      Return value = 232 - arg
```

For the program to print win, the condition (232 - arg) == 0 must be true.
Therefore, arg is 232.
232 in hexadecimal is `0xE8`
wrap in flag and pad with zero, and we have our answer `picoCTF{000000e8}`.


# Vault door 3
Well, let's see the source code
```java
achu@air ~ % cat /Users/achu/Downloads/VaultDoor3.java 
import java.util.*;

class VaultDoor3 {
    public static void main(String args[]) {
        VaultDoor3 vaultDoor = new VaultDoor3();
        Scanner scanner = new Scanner(System.in);
        System.out.print("Enter vault password: ");
        String userInput = scanner.next();
	String input = userInput.substring("picoCTF{".length(),userInput.length()-1);
	if (vaultDoor.checkPassword(input)) {
	    System.out.println("Access granted.");
	} else {
	    System.out.println("Access denied!");
        }
    }

    // Our security monitoring team has noticed some intrusions on some of the
    // less secure doors. Dr. Evil has asked me specifically to build a stronger
    // vault door to protect his Doomsday plans. I just *know* this door will
    // keep all of those nosy agents out of our business. Mwa ha!
    //
    // -Minion #2671
    public boolean checkPassword(String password) {
        if (password.length() != 32) {
            return false;
        }
        char[] buffer = new char[32];
        int i;
        for (i=0; i<8; i++) {
            buffer[i] = password.charAt(i);
        }
        for (; i<16; i++) {
            buffer[i] = password.charAt(23-i);
        }
        for (; i<32; i+=2) {
            buffer[i] = password.charAt(46-i);
        }
        for (i=31; i>=17; i-=2) {
            buffer[i] = password.charAt(i);
        }
        String s = new String(buffer);
        return s.equals("jU5t_a_sna_3lpm18g947_u_4_m9r54f");
    }
}
achu@air ~ % 
```
Okay... Java isn't really my thing yet but it isn't hard to see what's going on here.
The """encryption""" should be fairly easy to reverse looking at the for-loops.
I'll do it in Python.

```py
achu@air ~ % python3
Python 3.13.0 (main, Oct  7 2024, 05:02:14) [Clang 15.0.0 (clang-1500.3.9.4)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> s = "jU5t_a_sna_3lpm18g947_u_4_m9r54f"
>>> password = [None] * 32
>>> for i in range(32):
...     if i < 8: x = i
...     elif 8 <= i < 16: x = 23 - i
...     elif 16 <= i < 32 and i % 2 == 0: x = 46 - i
...     else: x = i
...     password[x] = s[i]
...     
>>> "".join(password)
'jU5t_a_s1mpl3_an4gr4m_4_u_79958f'
>>> 
```
Okay, we'll wrap that around the flag boiler, to get to get the flag `picoCTF{jU5t_a_s1mpl3_an4gr4m_4_u_79958f}`, and it works!
