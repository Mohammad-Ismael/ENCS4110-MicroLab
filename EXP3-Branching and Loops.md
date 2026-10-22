
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

---

### Execution Trace & Verification in Keil

1. **At `CMP R0, #10`:**
* $7 - 10$ produces `0xFFFFFFFD`.
* In ARM subtraction, the Carry flag ($C$) acts as an inverted borrow: when a borrow is required (e.g., $7 < 10$), $C = 0$.
* APSR flags state: **`N=1, Z=0, C=0, V=0`**, exactly matching your comment.


2. **At `STOP`:**
* `R4` contains `0x00000001` (from `BLT`).
* `R5` contains `0x00000001` (from `BNE` on `TST`).
* `R6` contains `0x00000001` (from `BEQ` on `TEQ`).