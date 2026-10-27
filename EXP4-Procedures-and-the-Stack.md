# Procedures and the Stack

## Learning Objectives

After completing this experiment, you will be able to:

* Call and return from procedures using `BL` and `BX LR`.
* Apply the ARM Procedure Call Standard (AAPCS) to pass parameters, return results, and decide which registers must be preserved.
* Use `PUSH` and `POP` to save and restore callee-saved registers and `LR` on the full descending stack.
* Implement nested and recursive procedures that manage the stack correctly.

## Experiment Overview

This experiment builds on the branching and loop patterns from Experiment 3 to write modular assembly code. You will define procedures, follow the AAPCS calling convention for arguments, return values, and register preservation, and use the stack to save the return address and working registers across nested and recursive calls. By the end of this lab, you will be able to split a program into reusable procedures that call each other safely.

---

## Theoretical Background

### ARM Architecture Procedure Call Standard (AAPCS)

Procedures are reusable blocks of code that encapsulate a specific task. They promote modular design, code reuse, and clearer program structure. In ARM assembly, procedures are implemented using branch-and-link instructions along with register usage conventions defined by the ARM Architecture Procedure Call Standard (AAPCS).

A `procedure` (also called a subroutine, function, or method in high-level languages like C, C++, or Java) is a self-contained block of assembly code designed to perform a specific task.

The AAPCS is the set of rules that define how functions exchange data and how registers must be preserved during a procedure call:

* **R0–R3**: Hold the first four parameters. R0 also holds the return value. Caller-saved.
* **Stack**: Any additional parameters beyond the first four are passed on the stack.
* **R4–R11**: Must be preserved by the callee. If a procedure uses them, it must save and restore them.
* **R12 (IP)**: Intra-procedure-call scratch register. Caller-saved — a callee may freely overwrite it, so (like R0–R3) never assume it survives a BL.
* **SP (R13)**: Stack pointer, always points to the current top of the stack.
* **LR (R14)**: Link register holds the return address. Caller-saved.

*Note: Callees are the procedures being called, while callers are the ones calling the procedure.*

In accordance with your template guidelines, here is Section 4.1.2 converted directly to Markdown, with the code snippet expanded into a full, ready-to-run Keil ARM Cortex-M4 assembly program:

# Procedure Templates

A procedure is entered with a `BL` (branch-with-link) instruction, which stores the return address in the link register `LR`. The callee returns by branching to `LR` (e.g., `BX LR`). By the AAPCS, the first four arguments are passed in `R0`–`R3` and the primary return value is placed in `R0`.

## Basic Procedure Template

**Example:** simple procedure that expects two integers in `R0` and `R1` and returns their sum in `R0`.

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
        ; Load example arguments into R0 and R1
        MOV     R0, #5                ; First parameter = 5
        MOV     R1, #10               ; Second parameter = 10

        ; Call procedure
        BL      AddTwo                ; R0 will hold return value (15 / 0x0F)

STOP    B       STOP

; Listing 4.1: Basic procedure structure
AddTwo  PROC
        ADD     R0, R0, R1            ; return R0+R1 in R0
        BX      LR
        ENDP

        END

```

*Note: The `PROC` and `ENDP` directives help define the start and end of a procedure and they could be safely omitted.*

---
## Procedure with Preserved Registers

![AAPCS Registers Usage](./Figers/AAPCS%20Registers%20Usage.png)

When a procedure uses callee-saved registers (R4–R11), it must preserve their original values by saving them on the stack and restoring them before returning:

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
        ; Set up example parameters for ProcessArray
        LDR     R0, =SampleArray       ; Load address of array into R0 (1st parameter)
        MOV     R1, #5                 ; Pass array length into R1 (2nd parameter)

        ; Call procedure (AAPCS guarantees R4-R6 remain untouched after this call)
        BL      ProcessArray           ; Result stored in R0

STOP    B       STOP

; Listing 4.2: Procedure using preserved registers
ProcessArray PROC
        PUSH    {R4-R6, LR}            ; Save used registers and LR
        MOV     R4, R0                 ; Save array pointer
        MOV     R5, R1                 ; Save array length
        MOV     R6, #0                 ; Initialize counter

        ; Process array using R4, R5, R6...

        MOV     R0, R6                 ; Return counter value
        POP     {R4-R6, PC}            ; Restore registers and return
        ENDP

        AREA    MYDATA, DATA, READWRITE
SampleArray
        DCD     10, 20, 30, 40, 50

        END

```
---