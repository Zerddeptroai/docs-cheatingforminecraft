# Assembly Language Crash Course

## Table of Contents
1. [What is Assembly?](#what-is-assembly)
2. [Why Learn Assembly?](#why-learn-assembly)
3. [Registers](#registers)
4. [Basic Instructions](#basic-instructions)
5. [Memory Access](#memory-access)
6. [Control Flow](#control-flow)
7. [Functions & Calling Conventions](#functions--calling-conventions)
8. [x86-64 Deep Dive](#x86-64-deep-dive)
9. [ARM64 Deep Dive](#arm64-deep-dive)
10. [ARM32 Basics](#arm32-basics)
11. [Common Patterns](#common-patterns)
12. [Practical Examples](#practical-examples)

---

## What is Assembly?

Assembly language is the lowest-level human-readable programming language. It's a 1-to-1 mapping with machine code instructions.

### Low Level Example

**C Code:**
```c
int add_numbers(int a, int b) {
    return a + b;
}
```

**Compiled to Assembly (x86-64):**
```asm
add_numbers:
    add     edi, esi        ; edi += esi
    mov     eax, edi        ; eax = edi
    ret
```

**Machine Code:**
```
01 FE 89 C7 C3
```

---

## Why Learn Assembly?

✅ **Understand performance** - Compiler optimizations, bottlenecks
✅ **Security** - Reverse engineering, exploit development, vulnerability analysis
✅ **Systems programming** - Bootloaders, kernels, embedded systems
✅ **Debugging** - Step through code at the lowest level
✅ **Reverse engineering** - Understand what compiled code actually does
✅ **Optimization** - Write critical code sections in assembly

---

## Registers

### x86-64 Registers (64-bit)

A register is a tiny, ultra-fast storage location on the CPU.

```
General Purpose Registers (64-bit names, can access smaller portions):
┌────────────────────────────────────────────────────────────┐
│ 64-bit │ 32-bit │ 16-bit │ 8-bit (low) │ Purpose           │
├────────────────────────────────────────────────────────────┤
│ RAX    │ EAX    │ AX     │ AL          │ Return value      │
│ RBX    │ EBX    │ BX     │ BL          │ Callee-saved      │
│ RCX    │ ECX    │ CX     │ CL          │ Counter/arg       │
│ RDX    │ EDX    │ DX     │ DL          │ Data              │
│ RSI    │ ESI    │ SI     │ SIL         │ Source (arg)      │
│ RDI    │ EDI    │ DI     │ DIL         │ Destination (arg) │
│ RBP    │ EBP    │ BP     │ BPL         │ Base pointer      │
│ RSP    │ ESP    │ SP     │ SPL         │ Stack pointer     │
│ R8     │ R8D    │ R8W    │ R8B         │ Additional arg    │
│ R9     │ R9D    │ R9W    │ R9B         │ Additional arg    │
│ R10    │ R10D   │ R10W   │ R10B        │ Temp              │
│ R11    │ R11D   │ R11W   │ R11B        │ Temp              │
│ R12    │ R12D   │ R12W   │ R12B        │ Callee-saved      │
│ R13    │ R13D   │ R13W   │ R13B        │ Callee-saved      │
│ R14    │ R14D   │ R14W   │ R14B        │ Callee-saved      │
│ R15    │ R15D   │ R15W   │ R15B        │ Callee-saved      │
└────────────────────────────────────────────────────────────┘
```

**Key Concept:** RAX register contains 64 bits, but you can access:
- `RAX` - All 64 bits
- `EAX` - Lower 32 bits (upper 32 bits zeroed)
- `AX` - Lower 16 bits
- `AL` - Lowest 8 bits
- `AH` - Upper 8 bits of AX (bits 8-15)

**Example:**
```asm
mov rax, 0x0123456789ABCDEF

mov eax, 0x11111111    ; RAX becomes 0x0000000011111111
mov ax, 0x2222         ; RAX becomes 0x0000000011112222
mov al, 0x33           ; RAX becomes 0x0000000011112233
```

### ARM64 Registers (64-bit)

```
General Purpose Registers:
┌──────────────────────────────────────────────┐
│ 64-bit │ 32-bit │ Purpose                    │
├──────────────────────────────────────────────┤
│ X0     │ W0     │ Return value / Arg 1      │
│ X1     │ W1     │ Return value / Arg 2      │
│ X2     │ W2     │ Arg 3                     │
│ X3     │ W3     │ Arg 4                     │
│ X4     │ W4     │ Arg 5                     │
│ X5     │ W5     │ Arg 6                     │
│ X6     │ W6     │ Arg 7                     │
│ X7     │ W7     │ Arg 8                     │
│ X8     │ W8     │ Indirect result / temp    │
│ X9-X15 │ W9-W15 │ Temporary registers       │
│ X16-X17│ W16-W17│ IP registers / temp       │
│ X18    │ W18    │ Platform register         │
│ X19-X28│ W19-W28│ Callee-saved              │
│ X29    │ W29    │ Frame pointer (LR)        │
│ X30    │ W30    │ Link register (return)    │
│ SP     │ WSP    │ Stack pointer             │
│ PC     │        │ Program counter (read-only)
└──────────────────────────────────────────────┘
```

### ARM32 Registers (32-bit)

```
General Purpose Registers:
┌──────────────────────────────────────┐
│ Register │ Purpose                    │
├──────────────────────────────────────┤
│ R0-R3    │ Arguments / Return value   │
│ R4-R11   │ Callee-saved registers     │
│ R12 (IP) │ Intra-procedure call       │
│ R13 (SP) │ Stack pointer              │
│ R14 (LR) │ Link register (return addr)│
│ R15 (PC) │ Program counter            │
└──────────────────────────────────────┘
```

### CPU Flags (x86-64/ARM)

Flags register contains single-bit conditions:

| Flag | Meaning | Set When |
|------|---------|----------|
| ZF | Zero Flag | Result is zero |
| CF | Carry Flag | Unsigned overflow |
| SF | Sign Flag | Result is negative |
| OF | Overflow Flag | Signed overflow |
| PF | Parity Flag | Even number of bits |

---

## Basic Instructions

### x86-64 Instructions

#### MOV - Move (Copy)
```asm
mov rax, 0x1000         ; RAX = 0x1000
mov rax, rbx            ; RAX = RBX
mov rax, [0x1000]       ; RAX = value at address 0x1000
mov [0x1000], rax       ; Address 0x1000 = RAX
```

**Note:** You CAN'T do:
```asm
mov [0x1000], [0x2000]  ; INVALID - can't move memory to memory directly
```

#### ADD / SUB / MUL / DIV
```asm
add rax, rbx            ; RAX += RBX
sub rax, 0x100          ; RAX -= 0x100
imul rax, rbx, 0x10     ; RAX = RBX * 0x10 (signed multiply)
div rcx                 ; RAX = RDX:RAX / RCX (64-bit / 64-bit)
```

**DIV Example:**
```asm
mov rax, 100            ; Dividend
mov rcx, 10             ; Divisor
div rcx                 ; RAX = 100/10 = 10, RDX = 100%10 = 0
```

#### INC / DEC
```asm
inc rax                 ; RAX++
dec rax                 ; RAX--
add rax, 1              ; Same as INC
```

#### AND / OR / XOR / NOT
```asm
and rax, rbx            ; RAX = RAX & RBX (bitwise AND)
or  rax, rbx            ; RAX = RAX | RBX (bitwise OR)
xor rax, rbx            ; RAX = RAX ^ RBX (bitwise XOR)
not rax                 ; RAX = ~RAX (bitwise NOT)
```

**Useful trick - Zero a register:**
```asm
xor eax, eax            ; EAX = 0 (faster than mov eax, 0)
```

#### LEA - Load Effective Address
```asm
lea rax, [rsi + rdi*2 + 0x100]  ; RAX = RSI + RDI*2 + 0x100
                                 ; (calculates address, doesn't load)
```

**Difference:**
```asm
mov rax, [0x1000]       ; Load VALUE at 0x1000 into RAX
lea rax, [0x1000]       ; Load ADDRESS 0x1000 into RAX
```

#### CMP - Compare
```asm
cmp rax, rbx            ; Compare RAX with RBX (sets flags)
                        ; Doesn't modify either register
```

### ARM64 Instructions

#### MOV - Move
```asm
mov x0, x1              ; X0 = X1
mov x0, 0x1000          ; X0 = 0x1000
mov w0, w1              ; W0 = W1 (32-bit)
```

#### ADD / SUB / MUL / DIV
```asm
add x0, x1, x2          ; X0 = X1 + X2
sub x0, x1, 0x100       ; X0 = X1 - 0x100
mul x0, x1, x2          ; X0 = X1 * X2
udiv x0, x1, x2         ; X0 = X1 / X2 (unsigned)
sdiv x0, x1, x2         ; X0 = X1 / X2 (signed)
```

#### AND / OR / EOR (XOR) / MVN (NOT)
```asm
and x0, x1, x2          ; X0 = X1 & X2
orr x0, x1, x2          ; X0 = X1 | X2
eor x0, x1, x2          ; X0 = X1 ^ X2 (EOR = XOR)
mvn x0, x1              ; X0 = ~X1
```

#### LSL / LSR / ASR - Shifts
```asm
lsl x0, x1, 2           ; X0 = X1 << 2 (logical shift left)
lsr x0, x1, 2           ; X0 = X1 >> 2 (logical shift right)
asr x0, x1, 2           ; X0 = X1 >> 2 (arithmetic shift right)
```

#### CMP - Compare
```asm
cmp x0, x1              ; Compare X0 with X1 (sets flags)
```

---

## Memory Access

### x86-64 Memory Addressing

```asm
[base + index*scale + displacement]

mov rax, [rsi]                          ; Load from RSI
mov rax, [rsi + 0x100]                  ; Load from RSI + 0x100
mov rax, [rsi + rdi*2]                  ; Load from RSI + RDI*2
mov rax, [rsi + rdi*4 + 0x100]          ; Load from RSI + RDI*4 + 0x100
```

**Scale:** Can be 1, 2, 4, or 8 (array indexing)

### ARM64 Memory Addressing

```asm
[base]                  ; Simple offset
[base, offset]          ; Base + offset
[base, index]           ; Base + index

ldr x0, [x1]            ; Load from X1
ldr x0, [x1, 0x100]     ; Load from X1 + 0x100
ldr x0, [x1, x2]        ; Load from X1 + X2
```

### Size Modifiers

**x86-64:**
```asm
mov al, [rsi]           ; 8-bit load
mov ax, [rsi]           ; 16-bit load
mov eax, [rsi]          ; 32-bit load
mov rax, [rsi]          ; 64-bit load
```

**ARM64:**
```asm
ldrb w0, [x1]           ; 8-bit load
ldrh w0, [x1]           ; 16-bit load
ldr w0, [x1]            ; 32-bit load
ldr x0, [x1]            ; 64-bit load
```

---

## Control Flow

### Jumps and Branches

#### x86-64 Jumps

```asm
jmp address             ; Unconditional jump

; Conditional jumps (based on flags)
je address              ; Jump if Equal (ZF set)
jne address             ; Jump if Not Equal (ZF clear)
jz address              ; Jump if Zero (same as JE)
jnz address             ; Jump if Not Zero (same as JNE)
jg address              ; Jump if Greater (signed)
jl address              ; Jump if Less (signed)
ja address              ; Jump if Above (unsigned)
jb address              ; Jump if Below (unsigned)
jo address              ; Jump if Overflow (OF set)
```

**Complete condition list:**
| Instruction | Flag Condition | When Set |
|-------------|---|---|
| JE / JZ | ZF | Result zero |
| JNE / JNZ | !ZF | Result not zero |
| JG | SF == OF | Signed greater |
| JL | SF != OF | Signed less |
| JGE | SF == OF | Signed >= |
| JLE | SF != OF \| ZF | Signed <= |
| JA | !CF & !ZF | Unsigned greater |
| JB | CF | Unsigned less |
| JAE | !CF | Unsigned >= |
| JBE | CF \| ZF | Unsigned <= |
| JO | OF | Overflow |
| JNO | !OF | No overflow |
| JS | SF | Sign bit set |
| JNS | !SF | Sign bit clear |

#### ARM64 Branches

```asm
b address               ; Unconditional branch

; Conditional branches (based on flags)
beq address             ; Branch if Equal (ZF set)
bne address             ; Branch if Not Equal (ZF clear)
bgt address             ; Branch if Greater (signed)
blt address             ; Branch if Less (signed)
bge address             ; Branch if >= (signed)
ble address             ; Branch if <= (signed)
bhi address             ; Branch if Higher (unsigned)
blo address             ; Branch if Lower (unsigned)
bhs address             ; Branch if Higher or Same (unsigned)
bls address             ; Branch if Lower or Same (unsigned)
bvs address             ; Branch if Overflow Set
bvc address             ; Branch if Overflow Clear
```

#### ARM32 Branches

```asm
b address               ; Unconditional branch
beq address             ; Branch if Equal
bne address             ; Branch if Not Equal
; Similar to ARM64
```

### Conditional Flow Example

**C Code:**
```c
if (x > 5) {
    y = 10;
} else {
    y = 20;
}
```

**x86-64 Assembly (x in RAX, y in RBX):**
```asm
cmp rax, 5              ; Compare RAX with 5
jle else_branch         ; Jump if <= 5
mov rbx, 10             ; y = 10
jmp end                 ; Jump to end
else_branch:
mov rbx, 20             ; y = 20
end:
```

**ARM64 Assembly (x in X0, y in X1):**
```asm
cmp x0, 5               ; Compare X0 with 5
ble else_branch         ; Branch if <=
mov x1, 10              ; y = 10
b end                   ; Branch to end
else_branch:
mov x1, 20              ; y = 20
end:
```

---

## Functions & Calling Conventions

### x86-64 System V ABI (Linux/Unix)

**Argument Passing:**
```
Integer arguments (1st to 6th):
RDI, RSI, RDX, RCX, R8, R9

Floating-point arguments:
XMM0 - XMM7

Extra arguments:
Passed on stack

Return value:
RAX (or RDX:RAX for 128-bit)
```

**Registers:**
- **Caller-saved** (Can be destroyed by called function): RAX, RCX, RDX, RSI, RDI, R8-R11
- **Callee-saved** (Must be preserved): RBX, RBP, R12-R15

**Function Prologue:**
```asm
push rbp                ; Save old base pointer
mov rbp, rsp            ; Set new base pointer
sub rsp, 0x20           ; Allocate 32 bytes for locals
```

**Function Epilogue:**
```asm
mov rsp, rbp            ; Restore stack pointer
pop rbp                 ; Restore base pointer
ret                     ; Return to caller
```

### x86-64 Function Example

**C Code:**
```c
int add(int a, int b, int c) {
    return a + b + c;
}
```

**x86-64 Assembly:**
```asm
add:
    add edi, esi        ; EDI = EDI + ESI (a + b)
    add edi, edx        ; EDI = EDI + EDX (result + c)
    mov eax, edi        ; EAX = EDI (return value)
    ret
```

### ARM64 Calling Convention

**Argument Passing:**
```
Integer arguments (1st to 8th):
X0, X1, X2, X3, X4, X5, X6, X7

Floating-point arguments:
D0-D7

Return value:
X0 (or X0:X1 for 128-bit)
```

**Registers:**
- **Caller-saved**: X0-X18, LR
- **Callee-saved**: X19-X28, SP, FP

**Function Prologue:**
```asm
stp x29, x30, [sp, -16]!    ; Save frame pointer and link register
mov x29, sp                  ; Set frame pointer
```

**Function Epilogue:**
```asm
ldp x29, x30, [sp], 16       ; Restore frame pointer and return address
ret                          ; Return to caller
```

### ARM64 Function Example

**C Code:**
```c
int add(int a, int b, int c) {
    return a + b + c;
}
```

**ARM64 Assembly:**
```asm
add:
    add w0, w0, w1          ; W0 = W0 + W1 (a + b)
    add w0, w0, w2          ; W0 = W0 + W2 (result + c)
    ret
```

---

## x86-64 Deep Dive

### Stack

The stack grows **downward** (from high addresses to low addresses).

```
High Address (0xFFFF...)
    ┌─────────────────┐
    │   Local vars    │
    ├─────────────────┤ ← RSP (after prologue)
    │   Return address│
    ├─────────────────┤
    │  Caller's frame │
    │     ...         │
Low Address (0x0000...)
```

### Push and Pop

```asm
push rax                ; RSP -= 8, [RSP] = RAX
pop rax                 ; RAX = [RSP], RSP += 8
```

**Equivalent to:**
```asm
sub rsp, 8              ; Subtract 8 from stack pointer
mov [rsp], rax          ; Store RAX on stack
```

### Stack Alignment

x86-64 requires **16-byte alignment** before a CALL instruction.

```asm
; Before CALL, RSP must be divisible by 16
; If RSP is misaligned, add padding
sub rsp, 8              ; Align RSP
call some_function
add rsp, 8              ; Restore
```

### CALL and RET

```asm
call function_address   ; Push return address, jump to function
ret                     ; Pop return address, jump to it
```

**Equivalent:**
```asm
call 0x1000
; is like:
push rip + next_instruction_length
jmp 0x1000
```

---

## ARM64 Deep Dive

### The Link Register (X30)

In ARM64, there's no dedicated return instruction encoded in the machine. Instead, the link register (X30) holds the return address.

```asm
bl function_address     ; Branch and Link
                        ; X30 = PC + 4
                        ; PC = function_address
```

**Return from function:**
```asm
ret                     ; Return using X30 (same as "br x30")
```

### Load/Store Pairs

ARM64 has special instructions for loading/storing two registers efficiently:

```asm
ldp x0, x1, [sp]        ; Load pair: X0 = [SP], X1 = [SP+8]
stp x0, x1, [sp, -16]!  ; Store pair with pre-index:
                        ; SP -= 16, [SP] = X0, [SP+8] = X1
ldp x29, x30, [sp], 16  ; Load pair with post-index:
                        ; X29 = [SP], X30 = [SP+8], SP += 16
```

### Immediate Values and MOVZ/MOVK

ARM64 can't load arbitrary 64-bit values with a single instruction:

```asm
mov x0, 0xFFFFFFFFFFFFFFFF  ; ERROR - too large

; Instead use MOVZ (Zero) + MOVK (Keep):
movz x0, 0x1234, lsl 0     ; X0 = 0x1234
movk x0, 0x5678, lsl 16    ; X0 = 0x56781234
movk x0, 0x90AB, lsl 32    ; X0 = 0x90AB56781234
movk x0, 0xCDEF, lsl 48    ; X0 = 0xCDEF90AB56781234
```

---

## ARM32 Basics

### Register Conventions

```asm
R0-R3   Arguments / Return
R4-R10  Callee-saved
R11     Frame pointer (FP)
R12     IP (Intra-procedure call)
R13     SP (Stack pointer)
R14     LR (Link register)
R15     PC (Program counter)
```

### Function Call

```asm
bl function_address     ; Branch and Link (call)
                        ; R14 = return address
```

### Return

```asm
bx lr                   ; Branch to Link Register (return)
```

### Function Example

**C Code:**
```c
int add(int a, int b) {
    return a + b;
}
```

**ARM32 Assembly:**
```asm
add:
    add r0, r0, r1      ; R0 = R0 + R1
    bx lr               ; Return
```

---

## Common Patterns

### Pattern 1: Loop

**C Code:**
```c
for (int i = 0; i < 10; i++) {
    // body
}
```

**x86-64:**
```asm
xor ecx, ecx            ; ECX = 0 (counter)
loop_start:
cmp ecx, 10             ; Compare with 10
jge loop_end            ; Jump if >= 10
; loop body
inc ecx                 ; i++
jmp loop_start
loop_end:
```

**ARM64:**
```asm
mov x0, 0               ; X0 = 0 (counter)
loop_start:
cmp x0, 10              ; Compare with 10
bge loop_end            ; Branch if >= 10
; loop body
add x0, x0, 1           ; i++
b loop_start
loop_end:
```

### Pattern 2: Array Access

**C Code:**
```c
int arr[10];
int x = arr[i];
```

**x86-64:**
```asm
mov rax, [rsi + rdi*4]  ; RAX = arr[i] (assuming arr in RSI, i in RDI, 4 bytes per int)
```

**ARM64:**
```asm
ldr w0, [x1, x2, lsl 2] ; W0 = arr[i] (X1 = arr, X2 = i, lsl 2 = *4)
```

### Pattern 3: Pointer Dereference

**C Code:**
```c
int *ptr = &x;
int y = *ptr;
```

**x86-64:**
```asm
mov rax, [rsi]          ; RAX = value at address in RSI
```

**ARM64:**
```asm
ldr x0, [x1]            ; X0 = value at address in X1
```

### Pattern 4: String Copy

**C Code:**
```c
strcpy(dest, src);
```

**x86-64 (simplified):**
```asm
xor ecx, ecx            ; Counter
copy_loop:
mov al, [rsi + rcx]     ; Load byte from src
mov [rdi + rcx], al     ; Store to dest
test al, al             ; Check if null terminator
jz copy_done
inc ecx
jmp copy_loop
copy_done:
```

### Pattern 5: Switch Statement (Jump Table)

**C Code:**
```c
switch(x) {
    case 0: y = 10; break;
    case 1: y = 20; break;
    case 2: y = 30; break;
}
```

**x86-64:**
```asm
cmp eax, 2              ; Check if x <= 2
ja default_case
lea rcx, [jump_table]   ; Load address of jump table
jmp [rcx + rax*8]       ; Jump based on x

jump_table:
dq case_0               ; offset 0
dq case_1               ; offset 8
dq case_2               ; offset 16

case_0:
mov rbx, 10
jmp end

case_1:
mov rbx, 20
jmp end

case_2:
mov rbx, 30

end:
```

---

## Practical Examples

### Example 1: Simple Addition Function

**C Code:**
```c
int add(int a, int b) {
    return a + b;
}
```

**x86-64:**
```asm
add:
    add edi, esi        ; EDI += ESI
    mov eax, edi        ; EAX = result
    ret
```

**ARM64:**
```asm
add:
    add w0, w0, w1
    ret
```

### Example 2: Conditional Return

**C Code:**
```c
int max(int a, int b) {
    if (a > b) return a;
    return b;
}
```

**x86-64:**
```asm
max:
    cmp edi, esi        ; Compare a with b
    jle else            ; Jump if a <= b
    mov eax, edi        ; Return a
    ret
else:
    mov eax, esi        ; Return b
    ret
```

**ARM64:**
```asm
max:
    cmp w0, w1          ; Compare a with b
    ble else            ; Branch if a <= b
    ret                 ; Return a (already in X0)
else:
    mov w0, w1          ; W0 = b
    ret
```

### Example 3: Loop - Sum Array

**C Code:**
```c
int sum_array(int *arr, int len) {
    int sum = 0;
    for (int i = 0; i < len; i++) {
        sum += arr[i];
    }
    return sum;
}
```

**x86-64:**
```asm
sum_array:
    xor eax, eax        ; sum = 0
    xor ecx, ecx        ; i = 0
    cmp esi, 0          ; Check len
    jle done
loop:
    add eax, [rdi + rcx*4]  ; sum += arr[i]
    inc ecx              ; i++
    cmp ecx, esi         ; i < len?
    jl loop
done:
    ret
```

**ARM64:**
```asm
sum_array:
    mov w2, 0           ; sum = 0
    mov w3, 0           ; i = 0
    cmp w1, 0           ; Check len
    ble done
loop:
    ldr w4, [x0, x3, lsl 2]  ; w4 = arr[i]
    add w2, w2, w4      ; sum += arr[i]
    add w3, w3, 1       ; i++
    cmp w3, w1          ; i < len?
    blt loop
    mov w0, w2          ; Return sum
done:
    ret
```

### Example 4: Recursive Function - Factorial

**C Code:**
```c
int factorial(int n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1);
}
```

**x86-64:**
```asm
factorial:
    push rbp            ; Save base pointer
    mov rbp, rsp        ; Set up stack frame
    cmp edi, 1          ; n <= 1?
    jg recursive
    mov eax, 1          ; Return 1
    pop rbp
    ret
recursive:
    push rdi            ; Save n
    sub edi, 1          ; n - 1
    call factorial      ; Recursive call
    pop rdi             ; Restore n
    imul eax, edi       ; EAX *= n
    pop rbp
    ret
```

**ARM64:**
```asm
factorial:
    stp x29, x30, [sp, -16]!  ; Save frame pointer and return address
    mov x29, sp               ; Set frame pointer
    cmp w0, 1                 ; n <= 1?
    bgt recursive
    mov w0, 1                 ; Return 1
    ldp x29, x30, [sp], 16
    ret
recursive:
    mov w9, w0                ; w9 = n (save it)
    sub w0, w0, 1             ; w0 = n - 1
    bl factorial              ; Recursive call, result in W0
    mul w0, w0, w9            ; w0 *= n
    ldp x29, x30, [sp], 16
    ret
```

---

## Tips for Reading Assembly

1. **Identify the function prologue/epilogue**
   - Prologue sets up the stack frame
   - Epilogue cleans it up

2. **Track register usage**
   - Follow where values come from and go to
   - Watch for argument passing

3. **Look for loops**
   - `cmp` followed by `j[condition]` backwards
   - `dec`/`inc` with jumps

4. **Identify function calls**
   - `call` (x86) or `bl` (ARM)
   - Arguments should be set up before the call

5. **Find branches**
   - `cmp` / `jcc` patterns show conditionals
   - Map them back to if/else in source

6. **Memory access**
   - Look for patterns with offsets
   - Arrays usually involve scaled indices

---

## Common Mistakes

❌ **Writing to both operands:**
```asm
mov rax, rbx            ; Correct
mov [rsi], [rdi]        ; WRONG - can't move memory to memory directly
```

✅ **Correct approach:**
```asm
mov rax, [rdi]          ; Load first
mov [rsi], rax          ; Store second
```

❌ **Forgetting to save/restore registers:**
```asm
; If you modify callee-saved registers, must restore
```

✅ **Correct approach:**
```asm
push rbx                ; Save
; Use RBX
pop rbx                 ; Restore
```

---

## Quick Reference

### x86-64 Common Operations

| Operation | Instruction |
|-----------|-------------|
| Zero a register | `xor eax, eax` |
| Move constant | `mov rax, 0x1000` |
| Add | `add rax, rbx` |
| Increment | `inc rax` |
| Loop counter | `xor ecx, ecx` then `loop:` ... `jmp loop` |
| Function return | `ret` |
| Conditional return | `je/jne/jg/jl` then `ret` |

### ARM64 Common Operations

| Operation | Instruction |
|-----------|-------------|
| Zero a register | `mov x0, 0` |
| Move constant | `mov x0, 0x1000` |
| Add | `add x0, x1, x2` |
| Increment | `add x0, x0, 1` |
| Loop counter | `mov x0, 0` then `loop:` ... `b loop` |
| Function return | `ret` |
| Branch | `b address` |
| Branch with link (call) | `bl address` |

---

## Practice Exercises

1. **Write assembly to add 3 numbers** (a in RAX, b in RBX, c in RCX)
2. **Write a loop that counts from 1 to 100**
3. **Write a function that swaps two values**
4. **Convert this C code to assembly:**
   ```c
   int factorial(int n) {
       int result = 1;
       for (int i = 2; i <= n; i++) {
           result *= i;
       }
       return result;
   }
   ```
5. **Write a simple string length function**

---

## Resources

- x86-64 Instruction Reference: `https://www.felixcloutier.com/x86/`
- ARM64 ISA Reference: ARM Official Documentation
- Assembly by Example: Build small programs and disassemble them
- Practice with Ghidra, Radare2, or IDA

---

*Keep practicing! Start with simple programs and work your way up to more complex code.*
