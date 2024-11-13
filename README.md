# Computer-Architecture-ncku2024fall
###### Assignment1: RISC-V Assembly and Instruction Pipeline
contributed by <[徐崇智](https://hackmd.io/@Jimmy-Xu)>
## [Quiz1](https://hackmd.io/@sysprog/arch2024-quiz1-sol) - Problem `C`
### C implementation of `__builtin_clz`
:::warning
In the case where `x = 0`, the result is undefined according to the GCC documentation, so we cannot allow 0 to be used as an input to the `my_clz` function.
:::
### Example
1. For `x = 1`
    * Binary representation of `1` is  `0000 0000 0000 0000 0000 0000 0000 0001`
    * The number of leading zero bits is `31`
    * Ouptut: `my_clz(1)` returns `31`
4. For `x = 255`
    * Binary representation of `255` is  `0000 0000 0000 0000 0000 0000 1111 1111`
    * The number of leading zero bits is `24`
    * Ouptut: `my_clz(255)` returns `24`
3. For `x = 536870911`
    * Binary representation of `536870911` is  `0001 1111 1111 1111 1111 1111 1111 1111`
    * There are no leading zeros.
    * Ouptut: `my_clz(536870911)` returns `3`
### C program

```c
#include <stdint.h>

int my_clz(uint32_t x) {
    int count = 0;
    for (int i = 31; i >= 0; i--) {
        if (x & (1U << i)) {
            break;
        }
        count++;
    }
    return count;
}
```

### Approach Explanation
#### Initialization
The function starts with a counter `count` initialized to zero, which will keep track of the number of leading zeros.
#### Bitwise operation
A for loop iterates from 31 down to 0, checking each bit position in the integer `x` . It uses a **logical left shift** operation `(1U << i)` to create a mask where only position i is set to 1, while all other positions are 0. This effectively allows for the detection of whether the specific bit in x is 1, enabling the calculation of the number of leading zeros.
#### Condition check
The condition `if (x & (1U << i))` checks if the i^th^ bit of `x` is 1. If it finds a 1, the loop break, however if the i^th^ bit is 0, it increments the count.
#### Return Value
When the loop is complete ( because a 1 was found or all bits have been checked ), the function returns the `count` , which represents the number of leading zeros in `x` .
### Assembly code (RISC-V)
```c
.data
argument1: .word   1
argument2: .word   255
argument3: .word   536870911
str1:      .string "The nearest power of two to "
str2:      .string " is "
new_line:  .string "\n"

.text
main:
    lw  a0, argument1           # Load the argument1 (1) into register a0
    jal ra, my_clz              # Jump-and-link to the 'my_clz' function to count leading zeros of argument1 (1)

    # Prepare to print the result
    lw  a1, argument1           # Reload the original argument into a1 to print it
    jal ra, printResult         # Call the function to print the result
    
    lw  a0, argument2           # Load the argument2 (255) into register a0
    jal ra, my_clz              # Jump-and-link to the 'my_clz' function to count leading zeros of argument2 (255)

    # Prepare to print the result
    lw  a1, argument2           # Reload the original argument into a1 to print it
    jal ra, printResult         # Call the function to print the result
    
    lw  a0, argument3           # Load the argument3 (536870911) into register a0
    jal ra, my_clz              # Jump-and-link to the 'my_clz' function to count leading zeros of argument3 (536870911)

    # Prepare to print the result
    lw  a1, argument3           # Reload the original argument into a1 to print it
    jal ra, printResult         # Call the function to print the result
    
    li a7, 10                   # Exit system call
    ecall

my_clz:
    addi t0, zero, 0            # Initialize t0 (count) to zero
    addi t1, zero, 31           # Set t1 = 31
loop:
    blt  t1, zero, exit         # Check if t1 < 0 then exit loop
    addi t2, zero, 1            # Set t2 = 1
    sll  t2, t2, t1             # Shift left logic on t2 by t1
    and  t3, a0, t2             # Do x & (1U << i)
    bne  t3, zero, exit         # if x & (1U << i) != 0 then exit loop
    addi t0, t0, 1              # count = count + 1
    addi t1, t1, -1             # t1 = t1 - 1 (i--)
    j    loop                   # goto loop label

exit:
    mv a0, t0                   # Move result from t0 to a0
    jr ra                       # Return to main

printResult:
    mv t0, a0                   # Save original input value (argument) in temporary register t0
    mv t1, a1                   # Save my_clz result in temporary register t1

    # Print "There are "
    la a0, str1                 # Load the address of the second string (" There are  ")
    li a7, 4                    # System call code for printing a string
    ecall

    # Print my_clz result (leading zero count)
    mv a0, t1                   # Move the `my_clz` result to a0 for printing
    li a7, 1                    # System call code for printing an integer
    ecall

    # Print " leading zeros in "
    la a0, str2                 # Load the address of the second string (" leading zeros in  ")
    li a7, 4                    # System call code for printing a string
    ecall

    # Print the original argument
    mv a0, t0                   # Move the input value (argument) to a0 for printing
    li a7, 1                    # System call code for printing an integer
    ecall
    
    # Print new_line
    la a0, new_line             # Load the address of the third string ("\n")
    li a7, 4                    # System call code for printing a string
    ecall

    jr ra                       # Return to main
```

### Output
Console
```
There are 31 leading zeros in 1
There are 24 leading zeros in 255
There are 3 leading zeros in 536870911
```

## Optimization -- Branchless Version 

### C Program
```c
#include <stdint.h>

int my_clz(uint32_t x) {
    int count = 0;

    if (x <= 0x0000FFFF) { count += 16; x <<= 16;} // check the first 16 bits
    if (x <= 0x00FFFFFF) { count += 8;  x <<= 8; } // check the first 8 bits
    if (x <= 0x0FFFFFFF) { count += 4;  x <<= 4; } // check the first 4 bits
    if (x <= 0x3FFFFFFF) { count += 2;  x <<= 2; } // check the first 2 bits
    if (x <= 0x7FFFFFFF) { count += 1;} // check the first 1 bits

    return count;
}
```
### Approach Explanation
#### Initialization
The function starts with a counter `count` initialized to zero, which will keep track of the number of leading zeros.
#### Condition Check
Instead of using a loop, it checks the most significant bits in chunks (16 bits, 8 bits, 4 bits, etc.), reducing the problem size with each step.
#### Bit Shifting
Once it’s determined that the leading bits are zeros, the value is shifted left by the corresponding number of bits to continue searching within the remaining bits.
#### Return Value
After the condition checks are complete, the function returns `count`, which indicates the number of leading zeros in `x` .

### Assembly Code (RISC-V)
```c
.data
argument1: .word   1
argument2: .word   255
argument3: .word   536870911
str1:      .string "The nearest power of two to "
str2:      .string " is "
new_line:  .string "\n"

.text
main:
    lw  a0, argument1           # Load the argument1 (1) into register a0
    jal ra, my_clz              # Jump-and-link to the 'my_clz' function to count leading zeros of argument1 (1)

    # Prepare to print the result
    lw  a1, argument1           # Reload the original argument into a1 to print it
    jal ra, printResult         # Call the function to print the result
    
    lw  a0, argument2           # Load the argument2 (255) into register a0
    jal ra, my_clz              # Jump-and-link to the 'my_clz' function to count leading zeros of argument2 (255)

    # Prepare to print the result
    lw  a1, argument2           # Reload the original argument into a1 to print it
    jal ra, printResult         # Call the function to print the result
    
    lw  a0, argument3           # Load the argument3 (536870911) into register a0
    jal ra, my_clz              # Jump-and-link to the 'my_clz' function to count leading zeros of argument3 (536870911)

    # Prepare to print the result
    lw  a1, argument3           # Reload the original argument into a1 to print it
    jal ra, printResult         # Call the function to print the result
    
    li a7, 10                   # Exit system call
    ecall

my_clz:
    addi t0, zero, 0            # Initialize t0 (count) to zero
    mv   t1, a0                 # Move a0 input (argument) into t1

    # Condition Check
    li   t2, 0x0000FFFF         # Load 0x0000FFFF into t2
    bleu t1, t2, step_16        # if x <= 0x0000FFFF then jump to step_16
    j    check_8                # jump to check_8

step_16:
    addi t0, t0, 16             # count += 16
    slli t1, t1, 16             # x <<= 16

check_8:
    li   t2, 0x00FFFFFF         # Load 0x00FFFFFF into t2
    bleu t1, t2, step_8         # if x <= 0x00FFFFFF then jump to step_8
    j    check_4                # jump to check_4

step_8:
    addi t0, t0, 8              # count += 8
    slli t1, t1, 8              # x <<= 8

check_4:
    li   t2, 0x0FFFFFFF         # Load 0x0FFFFFFF into t2
    bleu t1, t2, step_4         # if x <= 0x0FFFFFFF then jump to step_4
    j    check_2                # jump to check_2

step_4:
    addi t0, t0, 4              # count += 4
    slli t1, t1, 4              # x <<= 4

check_2:
    li   t2, 0x3FFFFFFF         # Load 0x3FFFFFFF into t2
    bleu t1, t2, step_2         # if x <= 0x3FFFFFFF then jump to step_2
    j    check_1                # jump to check_1

step_2:
    addi t0, t0, 2              # count += 2
    slli t1, t1, 2              # x <<= 2

check_1:
    li   t2, 0x7FFFFFFF         # Load 0x7FFFFFFF into t2
    bleu t1, t2, step_1         # if x <= 0x7FFFFFFF then jump to step_1
    j    exit                   # jump to exit

step_1:
    addi t0, t0, 1              # count += 1

exit:
    mv a0, t0                   # Move result from t0 to a0
    jr ra                       # Return to main

printResult:
    mv t0, a0                   # Save original input value (argument) in temporary register t0
    mv t1, a1                   # Save my_clz result in temporary register t1

    # Print "There are "
    la a0, str1                 # Load the address of the second string (" There are  ")
    li a7, 4                    # System call code for printing a string
    ecall

    # Print my_clz result (leading zero count)
    mv a0, t1                   # Move the `my_clz` result to a0 for printing
    li a7, 1                    # System call code for printing an integer
    ecall

    # Print " leading zeros in "
    la a0, str2                 # Load the address of the second string (" leading zeros in  ")
    li a7, 4                    # System call code for printing a string
    ecall

    # Print the original argument
    mv a0, t0                   # Move the input value (argument) to a0 for printing
    li a7, 1                    # System call code for printing an integer
    ecall
    
    # Print new_line
    la a0, new_line             # Load the address of the third string ("\n")
    li a7, 4                    # System call code for printing a string
    ecall

    jr ra                       # Return to main
```

### Instruction Explanation & Hazard Resolution
#### pseudo-instruction
1. `mv rd, rs`
    It translates to `addi rd, rs, 0` .
2. `j offset`
    It translates to `jal x0, offset` .

3. `jr rs`
    It translates to `jalr x0, 0(rs)` .
4. `li rd, immediate` 
    It is composed of two actual instructions: `lui` and `addi` . 
    
    For example, the pseudo-instruction `li t2, 0x0000FFFF` is translated into the following two instructions:

    1. `lui t2, 0x1` -> t2: 0x00010000
    2. `addi t2, t2, -1` -> t2: 0x0000FFFF

5. `bleu rs, rt, offset` 
    It translates to `bgeu` (branch if greater than or equal, unsigned) with the positions of rt and rs swapped.

    For example, the pseudo-instruction `bleu  t1, t2, step_16` is translated into `bgeu t2, t1, step_16` .

##### Result in [Ripes](https://ripes.me) simulator
> I think there is an error in [Ripes](https://ripes.me). The instruction `lui x7, 0x10` should actually be `lui x7, 0x1` .

![截圖 2024-10-12 下午2.51.33](https://hackmd.io/_uploads/SkSFn5Dykl.png)
:::info
In the image above, you can see that the instruction `li t2, 0x0000FFFF` is translated into `lui x7, 0x1` and `addi x7, x7, -1` . Similarly, the instruction `bleu t1, t2, step_16` is translated into `bgeu x7, x6, 8<step_16>` .
:::

> Reference: [RISC-V pseudoinstructions](https://riscv.org/wp-content/uploads/2019/06/riscv-spec.pdf) page 139

#### 5-Stages RISC-V Processor
We used [Ripes](https://ripes.me) to simulate a 5-Stage RISC-V Processor
![Pipeline](https://hackmd.io/_uploads/rJcZRlIkJl.png)
#### Control Hazard
At the 4^th^ clock cycle (C4), a **control hazard** (also known as a **branch hazard**) occurs, since the branch outcome is unknown, we must flush two instructions in the Instruction Fetch (IF) and Instruction Decode (ID) stages to prevent incorrect execution. This process is repeated for each subsequent branch instruction.

| Instruction |C1|C2|C3|C4|C5|C6|C7|C8|C9|
|---|---|---|---|---|---|---|---|---|---|
| lw   x10, 0(x10) |IF|ID|EXE|MEM|WB|||||
| jal  x1, 84 <my_clz> ||IF|ID|EXE|MEM|WB||||
| addi x5, x0 0 |||stall|stall|IF|ID|EXE|MEM|WB|

![Control Hazard](https://hackmd.io/_uploads/HJwtyGIkyx.png)
:::info
As shown in the picture above, there are two `nop` instructions present to resolve the control hazard.
:::
#### RAW (Read After Write) Data Hazard
In the following list, the **RAW (Read After Write) data hazard** is highlighted in <font color="#f00">red</font> and <font color="#0000ff">blue</font>. To resolve this issue, a hardware component known as the **forwarding unit** is required. This unit connects across pipeline stages and controls the multiplexer in the execution stage, which selecting the correct value to forward to the ALU. By forwarding the result from either the memory or write-back stage. The **forwarding unit** ensures that the result of the last executed instruction is immediately available.

| Instruction |C5|C6|C7|C8|C9|C10|C11|C12|C13|
|---|---|---|---|---|---|---|---|---|---|
| addi x5, x0 0 |IF|ID|EXE|MEM|WB|||||
| addi x6, x10, 0 ||IF|ID|EXE|MEM|WB||||
| lui <font color="#f00">x7</font>, 0x01 |||IF|ID|EXE|MEM|WB|||
| addi, <font color="#0000FF">x7</font>, <font color="#f00">x7</font>, -1 ||||IF|ID|EXE|MEM|WB||
| bgeu <font color="#0000FF">x7</font> x6 8 <step_16> |||||IF|ID|EXE|MEM|WB|
##### schematic
The left picture below illustrates the RISC-V architecture with a forwarding circuit, but branch detection has not been moved earlier to the execution stage. Typically, moving branch detection to an earlier stage, such as the execution stage, would require flushing or stalling only two instructions (or clock cycles) in the IF and ID stages, respectively, upon a branch misprediction. This optimization can significantly reduce the number of wasted cycles, improving overall execution efficiency by minimizing the performance penalties associated with control hazards.

In the right picture, the value of x7 becomes immediately available in the memory stage and is forwarded to the execution stage as the source value for the next instruction.

![RISC-Vcircuit](https://hackmd.io/_uploads/SJbvQ5Pk1e.png =70%x)![Forwarding](https://hackmd.io/_uploads/BJ2-ouPykl.png =30%x)

#### Execution information
Original Version:
![EXE info original](https://hackmd.io/_uploads/S1DyFUKJ1g.png)


Branchless Version:
![EXE info branchless](https://hackmd.io/_uploads/SJ-juItJJl.png)

:::info
You can clearly see that the cycle count for the branchless version is lower than that of the original version.
:::
## Use-case in [GeeksforGeeks](https://www.geeksforgeeks.org)

Here’s a problem that involves using the `clz`  (Count Leading Zeros) function: [Smallest power of 2 greater than or equal to n](https://www.geeksforgeeks.org/problems/smallest-power-of-2-greater-than-or-equal-to-n2630/0)

> Description: Given a number `n` .Find the nearest number which is a power of 2 and greater than or equal to `n` .
> 
>Constraints: 1 <= `n` <= 10^9^
### Example
1. For `x = 1`
    * The nearest power of 2 greater than or 
equal to 1 is 2^0^ = 1.
    * Ouptut: `nearestPowerOfTwo(1)` returns `1`
4. For `x = 5`
    * The nearest power of 2 greater than or 
equal to 1 is 2^3^ = 8.
    * Ouptut: `nearestPowerOfTwo(5)` returns `8`
3. For `x = 255`
    * The nearest power of 2 greater than or 
equal to 1 is 2^8^ = 256.
    * Ouptut: `nearestPowerOfTwo(255)` returns `256`
### Approach

1.	Use `clz` to determine the number of leading zeros in the binary representation of `n` .
2.	Based on the number of leading zeros, calculate the smallest power of 2 greater than or equal to `n` .

### C Program
:::info
Since the maximum value of `n` is limited by the constraints of what an `int` can store, I chose to use `int` as the return type for the function.
:::
```c
#include <stdint.h>

int my_clz(uint32_t x) {
    int count = 0;
    for (int i = 31; i >= 0; i--) {
        if (x & (1U << i)) {
            break;
        }
        count++;
    }
    return count;
}

int nearestPowerOfTwo(uint32_t n) {
    // If n is already a power of two, return n directly
    if ((n & (n - 1)) == 0) return n;

    // Use my_clz to count the leading zeros of n
    int leading_zeros = my_clz(n);

    // Calculate the smallest power of 2 greater than or equal to n
    int result = 1U << (32 - leading_zeros);

    return result;
}
```

:::warning
This problem in GeeksforGeeks is designed for a 64-bit system, but we are required to implement it on RV32I architecture. As a result, the C program must be adapted for a 32-bit system. If you attempt to use the following C program directly in GeeksforGeeks, it may result in errors due to this difference.
:::

### Assembly Code
```c
.data
argument1: .word   1
argument2: .word   5
argument3: .word   255
str1:      .string "The nearest power of two to "
str2:      .string " is "
new_line:  .string "\n"

.text
main:
    lw  a0, argument1           # Load the argument1 (1) into register a0
    jal ra, nearestPowerOfTwo   # Jump-and-link to the 'my_clz' function to count leading zeros of argument1 (1)

    # Prepare to print the result
    lw  a1, argument1           # Reload the original argument into a1 to print it
    jal ra, printResult         # Call the function to print the result
    
    lw  a0, argument2           # Load the argument2 (255) into register a0
    jal ra, nearestPowerOfTwo   # Jump-and-link to the 'my_clz' function to count leading zeros of argument2 (255)

    # Prepare to print the result
    lw  a1, argument2           # Reload the original argument into a1 to print it
    jal ra, printResult         # Call the function to print the result
    
    lw  a0, argument3           # Load the argument3 (536870911) into register a0
    jal ra, nearestPowerOfTwo   # Jump-and-link to the 'my_clz' function to count leading zeros of argument3 (536870911)

    # Prepare to print the result
    lw  a1, argument3           # Reload the original argument into a1 to print it
    jal ra, printResult         # Call the function to print the result
    
    li a7, 10                   # Exit system call
    ecall

my_clz:
    addi t0, zero, 0            # Initialize t0 (count) to zero
    mv   t1, a0                 # Move a0 input (argument) into t1

    # Condition Check
    li   t2, 0x0000FFFF         # Load 0x0000FFFF into t2
    bleu t1, t2, step_16        # if x <= 0x0000FFFF then jump to step_16
    j    check_8                # jump to check_8

step_16:
    addi t0, t0, 16             # count += 16
    slli t1, t1, 16             # x <<= 16

check_8:
    li   t2, 0x00FFFFFF         # Load 0x00FFFFFF into t2
    bleu t1, t2, step_8         # if x <= 0x00FFFFFF then jump to step_8
    j    check_4                # jump to check_4

step_8:
    addi t0, t0, 8              # count += 8
    slli t1, t1, 8              # x <<= 8

check_4:
    li   t2, 0x0FFFFFFF         # Load 0x0FFFFFFF into t2
    bleu t1, t2, step_4         # if x <= 0x0FFFFFFF then jump to step_4
    j    check_2                # jump to check_2

step_4:
    addi t0, t0, 4              # count += 4
    slli t1, t1, 4              # x <<= 4

check_2:
    li   t2, 0x3FFFFFFF         # Load 0x3FFFFFFF into t2
    bleu t1, t2, step_2         # if x <= 0x3FFFFFFF then jump to step_2
    j    check_1                # jump to check_1

step_2:
    addi t0, t0, 2              # count += 2
    slli t1, t1, 2              # x <<= 2

check_1:
    li   t2, 0x7FFFFFFF         # Load 0x7FFFFFFF into t2
    bleu t1, t2, step_1         # if x <= 0x7FFFFFFF then jump to step_1
    j    exit_my_clz            # jump to exit_my_clz

step_1:
    addi t0, t0, 1              # count += 1

exit_my_clz:
    mv a0, t0                   # Move result from t0 to a0
    jr ra                       # Return to main

nearestPowerOfTwo:
    addi sp, sp, -8             # Allocate stack space for local variables (ra)
    sw   ra, 0(sp)              # Save return address (ra) on the stack
    mv   t0, a0                 # Move argument into t0
    addi t1, t0, -1             # n - 1
    and  t2, t0, t1             # n & (n - 1)
    beq  t2, zero, exit         # if t2 == 0 then return n
    jal  ra, my_clz             # Store the result of my_clz(n) in a0
    mv   t0, a0                 # Move the result of my_clz(n) into t0
    sub  t0, zero, t0           # Store the negation of my_clz(n) in t0
    addi t1, t0, 32             # 32 - my_clz(n)
    addi t2, zero, 1            # Set t2 as 1U
    sll  t0, t2, t1             # 1U << (32 - my_clz(n))

exit:
    mv   a0, t0                 # Move the result of nearestPowerOfTwo into a0
    lw   ra, 0(sp)              # Restore return address of main (ra)
    jr   ra                     # Return to main
printResult:
    mv t0, a0                   # Save original input value (argument) in temporary register t0
    mv t1, a1                   # Save my_clz result in temporary register t1

    # Print "There are "
    la a0, str1                 # Load the address of the second string (" There are  ")
    li a7, 4                    # System call code for printing a string
    ecall

    # Print my_clz result (leading zero count)
    mv a0, t1                   # Move the `my_clz` result to a0 for printing
    li a7, 1                    # System call code for printing an integer
    ecall

    # Print " leading zeros in "
    la a0, str2                 # Load the address of the second string (" leading zeros in  ")
    li a7, 4                    # System call code for printing a string
    ecall

    # Print the original argument
    mv a0, t0                   # Move the input value (argument) to a0 for printing
    li a7, 1                    # System call code for printing an integer
    ecall
    
    # Print new_line
    la a0, new_line             # Load the address of the third string ("\n")
    li a7, 4                    # System call code for printing a string
    ecall

    jr ra                       # Return to main
```
### Instruction Explanation
#### Procedure Call
Since the `nearestPowerOfTwo` function **calls another function** `my_clz`, we must allocate **stack** space to store the return address (ra) of the `main` function. This ensures that after the `my_clz` function finishes execution, the program can correctly return to the calling function `nearestPowerOfTwo`, and eventually back to `main`. Therefore, stack management is important for preserving the correct control flow in the program.

#### Output
Console
The nearest power of two to 1 is 1
The nearest power of two to 5 is 8
The nearest power of two to 255 is 256
