## Flow Control

Flow control instructions alter the sequential execution of instructions by changing the program counter (**PC**). These instructions enable the implementation of conditional statements, loops, and procedure calls that are fundamental to structured programming.

### Condition Evaluation

Before implementing control flow, we must understand how to evaluate conditions and set processor flags. ARM provides dedicated instructions for comparing values and testing bit patterns that update the condition flags without storing results.

#### Table 3.1: ARM Cortex-M4 Compare and Test Instructions

| Instruction | Internal Operation | Result Written to Register? | **N** (Negative) | **Z** (Zero) | **C** (Carry / Borrow) | **V** (Signed Overflow) | Typical Primary Purpose |
| --- | --- | --- | --- | --- | --- | --- | --- |
| **`CMP Rn, Operand2`** | `Rn - Operand2` | **No** (discarded) | Set if result bit 31 = `1` | Set if `Rn == Operand2` | Set if **no borrow** ($Rn \ge Operand2$, unsigned); `0` if borrow | Set if signed arithmetic overflow occurs | Arithmetic comparison (less, equal, greater) |
| **`CMN Rn, Operand2`** | `Rn + Operand2` | **No** (discarded) | Set if result bit 31 = `1` | Set if `Rn == -Operand2` | Set if unsigned addition carries (overflows past 32 bits) | Set if signed addition overflow occurs | Compare negative: tests equality/relationship to a negative value without negating first |
| **`TST Rn, Operand2`** | `Rn & Operand2` (Bitwise AND) | **No** (discarded) | Set if result bit 31 = `1` | Set if result is `0` (tested bit is `0`) | Preserved (or shifter carry out) | **Unaffected** | Checking whether specific bits are set/cleared |
| **`TEQ Rn, Operand2`** | `Rn ^ Operand2` (Bitwise XOR) | **No** (discarded) | Set if result bit 31 = `1` | Set if result is `0` (all bits match) | Preserved (or shifter carry out) | **Unaffected** | Equality / sign check without affecting `C` or `V` |

---

**Usage Notes:**

* These instructions only affect the condition flags ($N, Z, C, V$) — they do not store a result.
* `CMP`/`CMN` are arithmetic comparisons; `TST`/`TEQ` are logical bitwise comparisons.
* Many data-processing instructions can update flags by appending `S` (e.g., `ADDS`, `SUBS`).
* **Common use:** immediately followed by conditional branches such as `BEQ`, `BNE`, `BGT`, etc.
  

The following is the reference table showing how **`CMP`**, **`TST`**, and **`TEQ`** evaluate operands, how each updates the Program Status Register flags (**N, Z, C, V**), and which conditional branch instructions check them.

---

### Comparison & Test Instructions Summary

| Instruction | Internal Operation | Result Written to Register? | **N** (Negative) | **Z** (Zero) | **C** (Carry / Borrow) | **V** (Signed Overflow) | Typical Primary Purpose |
| --- | --- | --- | --- | --- | --- | --- | --- |
| **`CMP Rn, Operand2`** | `Rn - Operand2` | **No** (discarded) | Set if result bit 31 = `1` | Set if `Rn == Operand2` | Set if **no borrow** ($Rn \ge Operand2$, unsigned); `0` if borrow | Set if signed arithmetic overflow occurs | Arithmetic comparison (less, equal, greater) |
| **`TST Rn, Operand2`** | `Rn & Operand2` (Bitwise AND) | **No** (discarded) | Set if result bit 31 = `1` | Set if result is `0` (tested bit is `0`) | Preserved (or shifter carry out) | **Unaffected** | Checking whether specific bits are set/cleared |
| **`TEQ Rn, Operand2`** | `Rn ^ Operand2` (Bitwise XOR) | **No** (discarded) | Set if result bit 31 = `1` | Set if result is `0` (all bits match) | Preserved (or shifter carry out) | **Unaffected** | Equality / sign check without affecting `C` or `V` |

---

### Conditional Branches Quick Reference

| Condition Code | Meaning | Flag Test | Used With |
| --- | --- | --- | --- |
| **`BEQ`** | Equal / Result was Zero | $Z = 1$ | `CMP`, `TST`, `TEQ` |
| **`BNE`** | Not Equal / Result was Non-Zero | $Z = 0$ | `CMP`, `TST`, `TEQ` |
| **`BLT`** | Signed Less Than ($Rn < \text{Op2}$) | $N \ne V$ | `CMP` (Signed) |
| **`BGE`** | Signed Greater Than or Equal ($Rn \ge \text{Op2}$) | $N = V$ | `CMP` (Signed) |
| **`BGT`** | Signed Greater Than ($Rn > \text{Op2}$) | $Z = 0 \text{ and } N = V$ | `CMP` (Signed) |
| **`BLE`** | Signed Less Than or Equal ($Rn \le \text{Op2}$) | $Z = 1 \text{ or } N \ne V$ | `CMP` (Signed) |
| **`BLO` / `BCC**` | Unsigned Lower ($Rn < \text{Op2}$) | $C = 0$ (Borrow needed) | `CMP` (Unsigned) |
| **`BHS` / `BCS**` | Unsigned Higher or Same ($Rn \ge \text{Op2}$) | $C = 1$ (No borrow) | `CMP` (Unsigned) |
| **`BHI`** | Unsigned Higher ($Rn > \text{Op2}$) | $C = 1 \text{ and } Z = 0$ | `CMP` (Unsigned) |
| **`BLS`** | Unsigned Lower or Same ($Rn \le \text{Op2}$) | $C = 0 \text{ or } Z = 1$ | `CMP` (Unsigned) |

---

### How to Predict the Flag Values

* **For `CMP Rn, #imm` ($Rn - \text{imm}$):**
* If $Rn = \text{imm}$: $Z=1, C=1, N=0, V=0$.
* If $Rn < \text{imm}$ (unsigned): $C=0$ (indicates a borrow occurred).
* If $Rn < \text{imm}$ (signed, no overflow): $N=1, V=0 \rightarrow (N \ne V)$, taking `BLT`.


* **For `TST Rn, #mask` ($Rn \ \& \ \text{mask}$):**
* If the selected bit(s) are all `0`: Result is `0` $\rightarrow Z=1 \rightarrow$ takes `BEQ`.
* If any selected bit is `1`: Result is non-zero $\rightarrow Z=0 \rightarrow$ takes `BNE`.


* **For `TEQ Rn, Rm` ($Rn \oplus Rm$):**
* If every bit in $Rn$ matches $Rm$: Result is `0` $\rightarrow Z=1 \rightarrow$ takes `BEQ`.
* If any bit differs: Result is non-zero $\rightarrow Z=0 \rightarrow$ takes `BNE`.


## Conditional Branching

Branch instructions are the primary mechanism for implementing flow control in ARM assembly. They modify the program counter to jump to different parts of the code based on conditions or unconditionally.

### Table 3.2: ARM Cortex-M4 Branch Instructions

| Instr. | Syntax | Description / Usage |
| --- | --- | --- |
| **B** | `B label` | Unconditional branch to label (always jumps). |
| **B<cond>** | `B<cond> label` | Conditional branch based on flags. |
| **BL** | `BL label` | Branch with link: calls a subroutine, storing return address in `LR`. (Procedures are covered in Experiment 4.) |
| **BX** | `BX Rm` | Branch to address in register, often `BX LR` to return from a subroutine. |
| **CBZ** | `CBZ Rn, label` | Branch if `Rn == 0`. Example: `CBZ R0, Done`. |
| **CBNZ** | `CBNZ Rn, label` | Branch if `Rn != 0`. Example: loop until counter reaches zero. |

**CBZ/CBNZ Constraints:**

* **Register:** Operand must be a low register (`R0`–`R7`).
* **Range:** Branch is forward-only; the destination must be within 0–126 bytes after the instruction.
* **Flags:** Does not update condition flags ($N, Z, C, V$).

> **Note:** For backward or longer jumps, use `CMP`/`TST` with conditional branches (`BEQ`, `BNE`, `BGT`, etc.).

---


**Examples 1: Arithmetic comparison using CMP, Bit test using TST, Equality check using TEQ.**

```assembly
; PCB Used: EK-TM4C123GXL
; MCU Used: TM4C123GH6PM
; MPU Used: ARM Cortex M4F - ARMv7-M
        AREA    RESET, CODE, READONLY
        EXPORT  __Vectors
__Vectors
        DCD     0x20001000            ; Initial SP (example top-of-stack)
        DCD     Reset_Handler          ; Reset vector

        AREA    MYCODE, CODE, READONLY
        ENTRY
        EXPORT  Reset_Handler
Reset_Handler

; ==============================================================================
; 1) CMP Demonstration
; With R0 = 7: 7 - 10 = -3 (0xFFFFFFFD)
; Flags set: Z = 0 (non-zero), C = 0 (borrow occurred), N = 1 (negative), V = 0 (no signed overflow)
; ==============================================================================
        MOV     R0, #7
        CMP     R0, #10               ; R0 - 10 -> Z=0, C=0, N=1, V=0
        BLT     LessThan10            ; Signed check (N != V): branch taken
        BGE     GreaterOrEqual

LessThan10
        MOV     R4, #1                ; Flag: R4 = 1 confirms R0 < 10
        B       Test_TST

GreaterOrEqual
        MOV     R4, #2
        B       Test_TST

; ==============================================================================
; 2) TST Demonstration
; Performs bitwise AND (Rn & Operand2), discards result, updates Z and N flags.
; ==============================================================================
Test_TST
        MOV     R1, #0x12             ; Binary: 0001 0010b
        TST     R1, #0x10             ; Test bit 4: 0x12 AND 0x10 = 0x10 != 0
        BEQ     BitClear              ; Branch if result == 0 (Z == 1)
        BNE     BitSet                ; Branch if result != 0 (Z == 0): branch taken

BitClear
        MOV     R5, #0
        B       Test_TEQ

BitSet
        MOV     R5, #1                ; Flag: R5 = 1 confirms bit 4 was 1
        B       Test_TEQ

; ==============================================================================
; 3) TEQ Demonstration
; Performs bitwise XOR (Rn ^ Operand2), discards result, updates Z and N flags.
; ==============================================================================
Test_TEQ
        MOV     R2, #0x55             ; Binary: 0101 0101b
        MOV     R3, #0x55             ; Binary: 0101 0101b
        TEQ     R2, R3                ; 0x55 XOR 0x55 = 0x00 -> Sets Z = 1
        BEQ     ValuesEqual           ; Branch if equal (Z == 1): branch taken
        BNE     ValuesNotEqual

ValuesEqual
        MOV     R6, #1                ; Flag: R6 = 1 confirms values matched
        B       STOP

ValuesNotEqual
        MOV     R6, #0
        B       STOP

STOP    B       STOP                  ; Infinite loop
        END
```

### Execution Trace & Verification in Keil

1. **At `CMP R0, #10`:**
* $7 - 10$ produces `0xFFFFFFFD`.
* In ARM subtraction, the Carry flag ($C$) acts as an inverted borrow: when a borrow is required (e.g., $7 < 10$), $C = 0$.
* APSR flags state: **`N=1, Z=0, C=0, V=0`**, exactly matching your comment.


2. **At `STOP`:**
* `R4` contains `0x00000001` (from `BLT`).
* `R5` contains `0x00000001` (from `BNE` on `TST`).
* `R6` contains `0x00000001` (from `BEQ` on `TEQ`).


## Conditional Execution

ARM assembly supports conditional execution, where most instructions can be conditionally executed based on the current state of the condition flags. This feature allows for efficient implementation of conditional statements without explicit branching.

**Conditional Instruction Format:**

Most ARM instructions can be made conditional by appending a condition code suffix:

```assembly
OPCODE{cond} Rd, Rn, Operand2
```

**Examples:**

* `ADDEQ R0, R1, R2` — Add only if equal ($Z = 1$)
* `MOVNE R3, #10` — Move only if not equal ($Z = 0$)
* `SUBGT R4, R4, #1` — Subtract only if greater than (signed)

**Advantages of Conditional Execution:**

* **Performance:** Eliminates branch instructions for simple conditional operations.
* **Code density:** Reduces the number of instructions needed.
* **Atomic operations:** Multiple related conditional operations can be grouped.

**The IF Then (IT) Block:**

Cortex-M4 executes only Thumb-2 code, where a condition suffix on a non-branch instruction (`MOVGT`, `ADDEQ`, etc.) is valid **only inside an IT (If-Then) block**. IT makes the next one to four instructions conditional: each one is marked **T** (execute if the condition holds) or **E** (execute if it does not), and every instruction in the block must carry the matching condition suffix.

```assembly
; PCB Used: EK-TM4C123GXL
; MCU Used: TM4C123GH6PM
; MPU Used: ARM Cortex M4F - ARMv7-M
        AREA    RESET, CODE, READONLY
        EXPORT  __Vectors
__Vectors
        DCD     0x20001000            ; Initial SP (example top-of-stack)
        DCD     Reset_Handler          ; Reset vector

        AREA    MYCODE, CODE, READONLY
        ENTRY
        EXPORT  Reset_Handler
Reset_Handler

; ==============================================================================
; Explicit IT Block Demonstration
; Compares R3 and R2, then uses an ITE (If-Then-Else) block to update registers
; based on whether R3 > R2.
; ==============================================================================

        ; --- Test Case 1: R3 > R2 (True Path) ---
        MOV     R2, #10               ; R2 = 10
        MOV     R3, #25               ; R3 = 25
        MOV     R4, #99               ; R4 = 99 (Initial sentinel value)

        CMP     R3, R2                ; 25 - 10 = 15 -> Updates flags (Z=0, N=0, C=1, V=0)
        
        ITE     GT                    ; IT Block: Next instruction is Then (GT), second is Else (LE)
        MOVGT   R2, R3                ; True (25 > 10): Executed -> R2 = R3 = 25
        MOVLE   R4, #0                ; False: Skipped -> R4 remains 99

        ; --- Test Case 2: R3 <= R2 (False Path) ---
        MOV     R2, #50               ; Set R2 = 50 (larger than R3)
        CMP     R3, R2                ; 25 - 50 = -25 -> Updates flags (Z=0, N=1, C=0, V=0)

        ITE     GT                    ; IT Block: GT (Then), LE (Else)
        MOVGT   R2, R3                ; False (25 > 50): Skipped
        MOVLE   R4, #0                ; True (25 <= 50): Executed -> R4 = 0

STOP    B       STOP                  ; Infinite loop
        END
```

If you write a conditional instruction without an `IT`, the Keil assembler normally inserts one for you. Look for it in the **Disassembly** window when you single-step Example 1 below. Conditional branches (`BEQ`, `BGT`, etc.) are encoded differently and never need an `IT`.

## How Branch Instructions Work


Branch instructions change the flow of execution by modifying the **Program Counter (PC)**. When a branch is executed, the instruction encodes an offset which is added to the current value of the PC.

#### Offset Calculation

The branch instruction contains a signed immediate value (positive or negative). The processor adds this offset (aligned to halfword boundaries) to the current PC:

* **Positive offset:** Causes a **forward branch** (jump to a higher memory address, later in the program).
* **Negative offset:** Causes a **backward branch** (jump to a lower memory address, earlier in the program).

> **Example:**
> Suppose a branch instruction is located at address `0x100`, and the assembler encodes an immediate offset of `-0x08`. The effective target address will be:
> $$0x100 + 4 + (-0x08) = 0xFC$$
> 
This means the processor jumps backward to an earlier instruction. Such negative offsets are typically used to implement loops (e.g., repeat until zero).


```assembly
; PCB Used: EK-TM4C123GXL
; MCU Used: TM4C123GH6PM
; MPU Used: ARM Cortex M4F - ARMv7-M
        AREA    RESET, CODE, READONLY
        EXPORT  __Vectors
__Vectors
        DCD     0x20001000            ; Initial SP (example top-of-stack)
        DCD     Reset_Handler          ; Reset vector

        AREA    MYCODE, CODE, READONLY
        ENTRY
        EXPORT  Reset_Handler
Reset_Handler

; ==============================================================================
; DEMONSTRATION OF PC-RELATIVE BRANCH OFFSET MATH
; Formula: Target Address = Current Instruction Address + 4 + Offset
; ==============================================================================

        MOV     R0, #3                ; Initialize loop counter = 3

; ------------------------------------------------------------------------------
; Target Label Address: Let's assume this label sits at address Address_A
; ------------------------------------------------------------------------------
Target_Label
        SUBS    R0, R0, #1            ; Decrement loop counter and set flags

; ------------------------------------------------------------------------------
; Backward Branch Instruction (Negative Offset)
; When this instruction executes:
; 1. Instruction location = Address_B
; 2. Current PC value     = Address_B + 4  (due to Thumb-2 pipeline pre-fetch)
; 3. Offset encoded       = Negative value (distance back to Target_Label)
;
; Target_Label Address = Address_B + 4 + (-Offset)
; ------------------------------------------------------------------------------
        BNE     Target_Label          ; If R0 != 0 (Z=0), jump backward to Target_Label

        MOV     R1, #0x42             ; Executed only after R0 reaches 0

STOP    B       STOP                  ; Forward branch offset = -2 (jumps to itself)
        END
```

### Condition Codes

Conditional branches use condition codes that test the processor status flags ($N, Z, C, V$) set by previous instructions. These enable implementing high-level constructs like if-statements and loops.

#### Table 3.3: Common ARM Condition Codes

| Cond. | Meaning | Flag State / Description |
| --- | --- | --- |
| **EQ** | Equal | Execute if $Z = 1$. |
| **NE** | Not equal | Execute if $Z = 0$. |
| **CS / HS** | Carry set / Unsigned higher or same | Execute if $C = 1$. |
| **CC / LO** | Carry clear / Unsigned lower | Execute if $C = 0$. |
| **MI** | Minus (negative) | Execute if $N = 1$. |
| **PL** | Plus (non-negative) | Execute if $N = 0$. |
| **VS** | Overflow set | Execute if $V = 1$. |
| **VC** | Overflow clear | Execute if $V = 0$. |
| **HI** | Unsigned higher | Execute if $C = 1$ and $Z = 0$. |
| **LS** | Unsigned lower or same | Execute if $C = 0$ or $Z = 1$. |
| **GE** | Greater or equal (signed) | Execute if $N = V$. |
| **LT** | Less than (signed) | Execute if $N \neq V$. |
| **GT** | Greater than (signed) | Execute if $Z = 0$ and $N = V$. |
| **LE** | Less or equal (signed) | Execute if $Z = 1$ or $N \neq V$. |
| **AL** | Always | Always execute (default if no condition). |
| **NV** | Never | Reserved / do not use. |


## For Loop Structure

A typical `for` loop has the structure: **initialization**, **condition check**, **body execution**, and **increment/decrement**. This type of loop executes a known number of times.


```assembly
; PCB Used: EK-TM4C123GXL
; MCU Used: TM4C123GH6PM
; MPU Used: ARM Cortex M4F - ARMv7-M
        AREA    RESET, CODE, READONLY
        EXPORT  __Vectors
__Vectors
        DCD     0x20001000            ; Initial SP
        DCD     Reset_Handler          ; Reset vector

; Data section containing array
        AREA    M_DATA, DATA, READONLY
array   DCD     10, 20, 30, 40, 50    ; Array of 5 integers (32-bit each)
length  EQU     5                     ; Number of elements

; Code section
        AREA    MYCODE, CODE, READONLY
        ENTRY
        EXPORT  Reset_Handler
Reset_Handler

        ; Initialization
        MOV     R0, #0                ; R0 = i = 0
        MOV     R1, #0                ; R1 = sum = 0
        LDR     R3, =array            ; R3 = base address of array

for_start

        ; Condition check
        CMP     R0, #length           ; Compare i with length (5)
        BGE     for_end               ; Branch to for_end if i >= 5

        ; body execution
        ; Load array[i] using scaled register-offset: EA = R3 + (R0 * 4)
        LDR     R2, [R3, R0, LSL #2]  ; R2 = array[i]
        ADD     R1, R1, R2            ; R1 (sum) += R2

        ; Increment
        ADD     R0, R0, #1            ; i++
        B       for_start             ; Repeat

for_end
        ; R1 now contains sum = 10 + 20 + 30 + 40 + 50 = 150 (0x96)

STOP    B       STOP                  ; Infinite loop
        END

```


## While Loop Structure

While loops check the condition before executing the loop body, potentially executing zero times if the initial condition is false. This type of loop is useful when the number of iterations is not known in advance and depends on dynamic conditions.

### Complete Executable Code

Here is the complete runnable program for the **TM4C123GH6PM** board:

```assembly
; PCB Used: EK-TM4C123GXL
; MCU Used: TM4C123GH6PM
; MPU Used: ARM Cortex M4F - ARMv7-M
        AREA    RESET, CODE, READONLY
        EXPORT  __Vectors
__Vectors
        DCD     0x20001000            ; Initial SP
        DCD     Reset_Handler          ; Reset vector

; Data section containing null-terminated string
        AREA    M_DATA, DATA, READONLY
mystring DCB    "Hello World!", 0     ; Null-terminated string (12 characters + 1 null byte)

; Code section
        AREA    MYCODE, CODE, READONLY
        ENTRY
        EXPORT  Reset_Handler
Reset_Handler

        ; Initialization
        LDR     R0, =mystring         ; R0 = pointer to string
        MOV     R1, #0                ; R1 = character count = 0

while_start
        LDRB    R2, [R0], #1          ; Load byte into R2, post-increment R0 pointer by 1
        CMP     R2, #0                ; Check if byte is null terminator (\0)
        BEQ     while_end             ; Exit loop if null terminator reached

        ADD     R1, R1, #1            ; Increment count: R1++
        B       while_start           ; Repeat loop

while_end
        ; R1 now holds 12 (0x0C), the total length of "Hello World!"

STOP    B       STOP                  ; Infinite loop
        END

```