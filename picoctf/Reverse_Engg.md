# Vault Door 3

> This vault uses for-loops and byte arrays. The source code for this vault is here: VaultDoor3.java

## Solution:

This challenge involves reverse engineering a Java program that scrambles a password and checks if it matches a specific string. We need to work backwards from the scrambled result to find the original password.

First, I analyzed the `checkPassword` method which takes a 32-character password and scrambles it through four different loops:

1. **Loop 1**: Copies first 8 characters as it is
2. **Loop 2**: Copies characters 8-15 in reverse order from positions 15-8  
3. **Loop 3**: Copies every other character starting from position 16 using a formula
4. **Loop 4**: Copies the remaining odd-indexed characters as it is

The scrambled result must equal: `"jU5t_a_sna_3lpm18g947_u_4_m9r54f"`

To solve this, I created a reverse mapping - starting from the final scrambled string and figuring out what the original password must have been at each position.

I wrote a Python script to reverse the scrambling process:

```python
# The target scrambled string
buffer = "jU5t_a_sna_3lpm18g947_u_4_m9r54f"
password = [''] * 32

# Loop 1: buffer[i] = password[i] for i=0..7
for i in range(8):
    password[i] = buffer[i]

# Loop 2: buffer[i] = password[23-i] for i=8..15
for i in range(8, 16):
    password[23 - i] = buffer[i]

# Loop 3: buffer[i] = password[46-i] for i=16,18,20,...,30
for i in range(16, 32, 2):
    password[46 - i] = buffer[i]

# Loop 4: buffer[i] = password[i] for i=31,29,27,...,17
for i in range(31, 16, -2):
    password[i] = buffer[i]

# Combine all characters to get the password
result = ''.join(password)
print(f"Password: {result}")
print(f"Flag: picoCTF{{{result}}}")
```

When I ran this script, it gave me the original password before scrambling.

## Flag:

```
picoCTF{jU5t_a_s1mpl3_an4gr4m_4_u_79958f}
```

## Concepts learnt:

- **Reverse Engineering**: Working backwards from a known output to determine the original input
- **String Manipulation**: Understanding how characters are rearranged in memory
- **Index Mapping**: Tracking how positions change between original and scrambled data
- **Java charAt() method**: Extracting specific characters from strings by position

## Notes:

- The key insight was recognizing that each loop fills specific positions in the buffer from specific positions in the password
- I initially tried to solve this manually but found it much easier to write a script
- The scrambling process essentially creates an anagram of the original password with a specific pattern
- The flag format requires wrapping the password with `picoCTF{}`

## Resources:

- [Java String charAt() method documentation](https://docs.oracle.com/javase/8/docs/api/java/lang/String.html#charAt-int-)
- [Python string manipulation guide](https://docs.python.org/3/tutorial/introduction.html#strings)


# ARMssembly 1

> For what argument does this program print `win` with variables `83, 0 and 3? File: chall_1.S Flag format: picoCTF{XXXXXXXX} -> (hex, lowercase, no 0x, and 32 bits. ex. 5614267 would be picoCTF{0055aabb})

## Solution:

Let's analyze the ARM assembly code step by step. The key function is `func` which processes our input.

**Understanding the `func` function:**

1. **Variable Setup:**
   ```assembly
   mov	w0, 83        ; Store 83 in [sp, 16]
   str	wzr, [sp, 20] ; Store 0 in [sp, 20]  
   mov	w0, 3         ; Store 3 in [sp, 24]
   ```

2. **Operations:**
   ```assembly
   ldr	w0, [sp, 20]  ; Load 0 into w0
   ldr	w1, [sp, 16]  ; Load 83 into w1
   lsl	w0, w1, w0    ; Left shift: 83 << 0 = 83
   str	w0, [sp, 28]  ; Store result (83)
   ```

   ```assembly
   ldr	w1, [sp, 28]  ; Load 83 into w1
   ldr	w0, [sp, 24]  ; Load 3 into w0
   sdiv	w0, w1, w0    ; Signed division: 83 / 3 = 27
   str	w0, [sp, 28]  ; Store result (27)
   ```

   ```assembly
   ldr	w1, [sp, 28]  ; Load 27 into w1
   ldr	w0, [sp, 12]  ; Load our input into w0
   sub	w0, w1, w0    ; Subtract: 27 - input
   str	w0, [sp, 28]  ; Store result
   ```

3. **Win Condition in main:**
   ```assembly
   cmp	w0, 0         ; Compare result with 0
   bne	.L4           ; If not equal, jump to lose
   ```

**Mathematical Equation:**
The program prints "win" when:
```
27 - input = 0
```
Therefore:
```
input = 27
```

**Convert to Hexadecimal:**
27 in decimal = 0x1B in hexadecimal
As a 32-bit value with no 0x prefix: `0000001b`

## Flag:

```
picoCTF{0000001b}
```

## Concepts learnt:

- **ARM Assembly Analysis**: Understanding ARMv8 instruction set
- **Bit Shifting**: `lsl` instruction for left shifts
- **Division Operations**: `sdiv` for signed division
- **Stack Operations**: How local variables are stored on stack
- **Conditional Branching**: Using `cmp` and `bne` for flow control

## Notes:

- The `lsl w0, w1, w0` instruction performs: `w1 << w0`
- Since w0 was 0, `83 << 0 = 83` (no shift occurred)
- Integer division `83 / 3 = 27` (remainder discarded)
- The win condition requires the final result to be exactly 0
- The solution is the decimal value 27 converted to 32-bit hex

## Resources:

- [ARM Instruction Set Reference](http://infocenter.arm.com/help/index.jsp)
- [ARM Assembly Basics](https://azeria-labs.com/writing-arm-assembly-part-1/)
- [Hexadecimal Converter](https://www.rapidtables.com/convert/number/decimal-to-hex.html)


# GDB baby step 1

> Can you figure out what is in the `eax` register at the end of the `main` function? Put your answer in the picoCTF flag format: **picoCTF{n}** where n is the contents of the `eax` register in the decimal number base. If the answer was `0x11` your flag would be **picoCTF{17}.** Disassemble this.

## Solution:

This challenge requires using GDB (GNU Debugger) to examine a binary file and find what value is stored in the `eax` register when the main function ends.

**Step-by-step solution:**

1. **First, I navigated to the directory containing the binary file:**
   ```bash
   cd /mnt/d/picocTF/
   ```

2. **Made the file executable:**
   ```bash
   chmod +x debugger0_a
   ```

3. **Opened the file in GDB:**
   ```bash
   gdb ./debugger0_a
   ```

4. **Inside GDB, I disassembled the main function to see the assembly code:**
   ```bash
   (gdb) disassemble main
   ```

5. **The output showed the assembly code:**
   ```
   Dump of assembler code for function main:
      0x0000000000001129 <+0>:     endbr64
      0x000000000000112d <+4>:     push   %rbp
      0x000000000000112e <+5>:     mov    %rsp,%rbp
      0x0000000000001131 <+8>:     mov    %edi,-0x4(%rbp)
      0x0000000000001134 <+11>:    mov    %rsi,-0x10(%rbp)
      0x0000000000001138 <+15>:    mov    $0x86342,%eax
      0x000000000000113d <+20>:    pop    %rbp
      0x000000000000113e <+21>:    ret
   ```

6. **I identified the key instruction:**
   - `mov $0x86342,%eax` - This moves the hexadecimal value `0x86342` into the `eax` register

7. **Exited GDB and converted the hex value to decimal:**
   ```bash
   (gdb) quit
   echo $((0x86342))
   ```

8. **The conversion gave me: `549698`**

The value in the `eax` register at the end of main is `549698` in decimal.

## Flag:

```
picoCTF{549698}
```

## Concepts learnt:

- **GDB Usage**: How to use GNU Debugger to analyze binaries
- **x86_64 Assembly**: Understanding basic assembly instructions and registers
- **Disassembly**: Converting machine code back to readable assembly language
- **Register Analysis**: Identifying values stored in CPU registers
- **Hexadecimal to Decimal Conversion**: Converting between number bases

## Notes:

- The `eax` register is a 32-bit register used for return values in x86_64 architecture
- The `mov` instruction copies data into registers
- The value `0x86342` is in hexadecimal format (prefix `0x`)
- The main function's return value is stored in `eax` before the function ends
- GDB's `disassemble` command shows the assembly code of functions

## Resources:

- [GDB Quick Reference](https://users.ece.utexas.edu/~adnan/gdb-refcard.pdf)
- [x86_64 Assembly Guide](http://cs.lmu.edu/~ray/notes/x86assembly/)
- [Number Base Converter](https://www.rapidtables.com/convert/number/hex-to-decimal.html)
  
