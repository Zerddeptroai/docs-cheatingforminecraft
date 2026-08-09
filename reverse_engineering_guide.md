# Reverse Engineering: Educational Concepts & Techniques

## Table of Contents
1. [Fundamentals](#fundamentals)
2. [Binary Analysis](#binary-analysis)
3. [Common Tools](#common-tools)
4. [Techniques](#techniques)
5. [Memory Analysis](#memory-analysis)
6. [Dynamic Analysis](#dynamic-analysis)
7. [Legal & Ethical Considerations](#legal--ethical-considerations)

---

## Fundamentals

### What is Reverse Engineering?

Reverse engineering is the process of analyzing a system, application, or binary to understand its structure, functionality, and behavior. It involves working backwards from the compiled code to understand the original logic.

### Why Learn Reverse Engineering?

**Legitimate Applications:**
- **Security Research**: Finding vulnerabilities in software
- **Malware Analysis**: Understanding malicious code behavior
- **Interoperability**: Creating compatible software without source code
- **Legacy System Maintenance**: Understanding old systems without documentation
- **Education**: Learning how compilers and systems work
- **Incident Response**: Analyzing compromised systems
- **Game Modding**: Creating mods for games that permit it

---

## Binary Analysis

### Executable File Formats

#### ELF (Executable and Linkable Format) - Linux/Android
```
ELF Header
├── Magic Number (0x7F454C46)
├── Class (32-bit or 64-bit)
├── Data (Endianness)
├── OS/ABI
└── Entry Point

Program Headers (segments)
├── PT_LOAD (loadable segments)
├── PT_DYNAMIC (dynamic linking info)
├── PT_NOTE (notes)
└── PT_INTERP (interpreter)

Section Headers
├── .text (executable code)
├── .data (initialized data)
├── .bss (uninitialized data)
├── .rodata (read-only data)
├── .symtab (symbol table)
├── .strtab (string table)
├── .rel.* (relocation info)
└── .dynamic (dynamic linking structures)
```

#### PE (Portable Executable) - Windows
```
DOS Header
COFF Header
Optional Header
Section Headers (.text, .data, .rsrc, etc.)
```

### Sections Explained

| Section | Purpose | Writable |
|---------|---------|----------|
| `.text` | Machine code instructions | No (RX) |
| `.data` | Pre-initialized global variables | Yes (RW) |
| `.bss` | Uninitialized globals (zero-filled) | Yes (RW) |
| `.rodata` | String literals and constants | No (R) |
| `.symtab` | Symbol table (function/variable names) | N/A |
| `.strtab` | String table for symbols | N/A |
| `.dynsym` | Dynamic symbol table | N/A |
| `.got` | Global Offset Table (for relocations) | Yes (RW) |
| `.plt` | Procedure Linkage Table (dynamic calls) | No (RX) |

---

## Common Tools

### Static Analysis Tools

#### 1. **IDA Pro** (Interactive Disassembler)
- Industry standard disassembler
- Converts binary to assembly
- Supports many architectures
- Expensive but powerful
- Free version: IDA Free

```bash
# Command line usage
ida64 -B -S"idc_script.idc" binary_file
```

#### 2. **Ghidra** (NSA's Open Source Alternative)
```bash
# Free, open-source decompiler
ghidra_analyzeHeadless /path/to/project ProjectName -import /path/to/binary
```

#### 3. **Radare2** (Unix Reverse Engineering Framework)
```bash
r2 binary_file
# Interactive commands
> aaa          # Analyze all
> pdf @main    # Print disassembly of main
> s 0x1000     # Seek to address
> px 100       # Print hex
> ps 100       # Print string
```

#### 4. **Objdump** (GNU Binary Tools)
```bash
# Disassemble executable
objdump -d binary_file

# Display all headers
objdump -x binary_file

# Show symbol table
objdump -t binary_file
```

#### 5. **Strings** (Extract Readable Text)
```bash
# Find all printable strings
strings binary_file

# With offsets
strings -t x binary_file

# Specific strings
strings binary_file | grep "password"
```

#### 6. **Hexdump/xxd** (View Raw Bytes)
```bash
hexdump -C binary_file
xxd binary_file
```

### Dynamic Analysis Tools

#### 1. **GDB** (GNU Debugger)
```bash
# Start debugging
gdb ./program

# GDB commands
(gdb) break main              # Set breakpoint
(gdb) run                     # Start execution
(gdb) step                    # Single step (into functions)
(gdb) next                    # Single step (over functions)
(gdb) continue                # Continue execution
(gdb) print variable_name     # Print variable
(gdb) register info           # Show all registers
(gdb) memory read 0x1000 0x1100  # Read memory
(gdb) disassemble main        # Disassemble function
```

#### 2. **Strace** (System Call Tracing)
```bash
# Trace all system calls
strace ./program

# Follow child processes
strace -f ./program

# Output to file
strace -o trace.txt ./program

# Filter by category
strace -e trace=file ./program
```

#### 3. **Ltrace** (Library Call Tracing)
```bash
# Trace library function calls
ltrace ./program

# With timestamps
ltrace -t ./program
```

#### 4. **Frida** (Dynamic Instrumentation Toolkit)
```python
import frida

session = frida.attach("process_name")
script = session.create_script("""
Interceptor.attach(Module.findExportByName(null, "malloc"), {
    onEnter: function(args) {
        console.log("malloc called with size: " + args[0]);
    }
});
""")
script.load()
```

#### 5. **Android Studio Debugger** (For Android Apps)
- Set breakpoints in code
- Inspect variables and memory
- Step through code execution
- View logcat output

---

## Techniques

### 1. Static Disassembly

**Process:**
```
Binary File → Disassembler → Assembly Code → Manual Analysis
```

**Example - ARM64 Assembly:**
```asm
; Function prologue
stp     x29, x30, [sp, -16]!
mov     x29, sp

; Load address
adrp    x0, 0x400000          ; Address of page
add     x0, x0, #0x123        ; Add offset

; Function call
bl      0x1000                ; Branch and link (call)

; Return
ldp     x29, x30, [sp], 16
ret
```

### 2. Symbol Analysis

**Symbols provide clues about functionality:**

```bash
# Extract symbol information
readelf -s binary_file
nm binary_file

# Output example:
0000000000001000 T main
0000000000001100 T function_name
0000000000002000 d global_variable
                 U printf            # Undefined (imported)
```

**Symbol Types:**
- `T` - Text (code, executable)
- `D` - Data (initialized data)
- `B` - BSS (uninitialized data)
- `U` - Undefined (imported from library)
- `W` - Weak symbol

### 3. Control Flow Analysis

**Understanding program flow:**

```c
// Original code
if (condition) {
    function_a();
} else {
    function_b();
}

// Might compile to:
cmp     x0, 0           ; Compare x0 with 0
beq     else_branch     ; Branch if equal
bl      function_a      ; Call function_a
b       end             ; Unconditional jump
else_branch:
bl      function_b      ; Call function_b
end:
; Continue...
```

### 4. String and Resource Analysis

```bash
# Find all strings in binary
strings binary_file

# Strings with addresses
strings -t x binary_file

# Look for specific patterns
strings binary_file | grep -E "(http|password|key|token|api)"
```

### 5. Memory Layout Analysis

**Understanding runtime memory:**

```
High Address
┌─────────────────┐
│  Stack          │  (grows down)
├─────────────────┤
│  Heap           │  (grows up)
├─────────────────┤
│  BSS (.bss)     │  Uninitialized data
├─────────────────┤
│  Data (.data)   │  Initialized data
├─────────────────┤
│  Text (.text)   │  Code (read-only)
├─────────────────┤
│  Headers        │
└─────────────────┘
Low Address
```

---

## Memory Analysis

### Memory Protection (Linux)

**View memory mappings:**
```bash
# While process is running
cat /proc/[PID]/maps

# Example output:
# 55c8a2d000-55c8a2e000 r-xp 00000000 08:01 12345 /path/to/binary
# 55c8a3d000-55c8a3e000 rw-p 00000000 00:00 0
# 7f1234567000-7f1234569000 r-xp 00000000 08:01 67890 /lib/libc.so.6
```

**Permission Flags:**
- `r` - Read
- `w` - Write
- `x` - Execute
- `p` - Private (copy-on-write)
- `s` - Shared

### ASLR (Address Space Layout Randomization)

**Concept:** Libraries and code load at random addresses for security

```bash
# Disable ASLR (for testing/education)
echo 0 | sudo tee /proc/sys/kernel/randomize_va_space

# Re-enable
echo 2 | sudo tee /proc/sys/kernel/randomize_va_space

# Check ASLR status on binary
readelf -l binary_file | grep INTERP
```

### Finding Function Addresses

**Method 1: From Symbol Table**
```bash
nm binary_file | grep function_name
```

**Method 2: Disassembly**
```bash
objdump -d binary_file | grep -A 5 "function_name"
```

**Method 3: Dynamic (at runtime)**
```bash
# Using gdb
(gdb) info functions
(gdb) break function_name
```

**Method 4: Library Base + Offset**
```
Function Address = Library Base Address + Offset from Binary
```

Example:
```bash
# Find library base (ASLR randomized at runtime)
cat /proc/[PID]/maps | grep libc

# Output: 7f1234567000-7f1234589000 r-xp 00000000
# Base: 0x7f1234567000

# Find offset in binary
nm binary_file | grep function_name
# Offset: 0x1234

# Runtime address = 0x7f1234567000 + 0x1234 = 0x7f1234568234
```

---

## Dynamic Analysis

### Breakpoint Debugging

```bash
gdb ./program
(gdb) break *0x1000          # Break at address
(gdb) break main             # Break at function
(gdb) break file.c:10        # Break at line
(gdb) condition 1 x == 5     # Conditional breakpoint
```

### Watchpoints (Data Breakpoints)

```bash
(gdb) watch variable_name    # Break when modified
(gdb) rwatch variable_name   # Break when read
(gdb) awatch variable_name   # Break on any access
```

### Memory Inspection

```bash
# Read memory
(gdb) x /100x 0x1000        # 100 hex values
(gdb) x /50s 0x2000         # 50 strings
(gdb) x /10i 0x3000         # 10 instructions

# Format specifiers
# x - hex
# i - instruction
# s - string
# c - character
# d - decimal
```

### System Call Analysis

```bash
strace -e trace=open,read,write ./program
strace -e trace=network ./program
strace -e signal=SIGSEGV ./program
```

---

## Assembly Language Fundamentals

### Common Architectures

#### x86-64 (Intel/AMD 64-bit)
```asm
mov rax, rbx        ; Move (copy)
add rax, 0x100      ; Add
sub rax, rbx        ; Subtract
call 0x1000         ; Call function
ret                 ; Return
```

#### ARM64 (AArch64)
```asm
mov x0, x1          ; Move
add x0, x1, #0x100  ; Add
sub x0, x1, x2      ; Subtract
bl  0x1000          ; Branch and link (call)
ret                 ; Return
```

#### ARM (32-bit)
```asm
mov r0, r1          ; Move
add r0, r1, #0x100  ; Add
bl  0x1000          ; Branch and link
bx  lr              ; Return
```

### Calling Conventions

**x86-64 (System V AMD64 ABI):**
- First 6 integer args: `rdi, rsi, rdx, rcx, r8, r9`
- Return value: `rax`
- Caller cleans stack

**ARM64:**
- First 8 args: `x0-x7`
- Return value: `x0`

---

## Practical Exercise Examples

### Exercise 1: Basic Binary Analysis

```bash
# 1. Get file type
file program

# 2. Extract symbols
nm program | head -20

# 3. List sections
readelf -S program

# 4. View main function
objdump -d program | grep -A 20 "<main>"

# 5. Find strings
strings program | grep -i "error\|success"
```

### Exercise 2: Debugging with GDB

```bash
# Start debugging
gdb ./program

# Set breakpoint and run
(gdb) break main
(gdb) run argument1 argument2

# Step through
(gdb) step
(gdb) step
(gdb) next

# Inspect state
(gdb) info locals
(gdb) info registers
(gdb) print variable_name
(gdb) backtrace
```

### Exercise 3: System Call Tracing

```bash
# Trace file operations
strace -e trace=open,openat,read,write -o trace.txt ./program

# Analyze trace
grep "open" trace.txt
grep "read" trace.txt
```

### Exercise 4: Library Analysis

```bash
# Find linked libraries
ldd ./program

# Analyze library symbols
nm -D /lib/x86_64-linux-gnu/libc.so.6 | grep malloc

# Trace library calls
ltrace -c ./program
```

---

## Legal & Ethical Considerations

### When RE is Legal

✅ **Permitted uses:**
- Your own software (security research)
- Interoperability research (with restrictions)
- Education (on your systems)
- Security research (with authorization)
- Incident response (on systems you own)
- Competition/compatibility analysis (fair use)

### When RE is Illegal

❌ **Prohibited uses:**
- Circumventing copyright protection (DMCA in US)
- Reverse engineering without authorization on systems you don't own
- Creating unauthorized cheats/exploits
- Violating Terms of Service
- Stealing trade secrets
- Unauthorized access (hacking)

### Guidelines

1. **Know the law** in your jurisdiction
2. **Get permission** when reverse engineering systems you don't own
3. **Respect Terms of Service** agreements
4. **Don't distribute** circumvention tools or cracked software
5. **Use for legitimate purposes** (security, education, research)
6. **Publish responsibly** (responsible disclosure for vulnerabilities)

---

## Resources for Learning

### Books
- "Reverse Engineering for Beginners" - Dennis Yurichev (FREE)
- "The IDA Pro Book" - Chris Eagle
- "Assembly Language Step-by-Step" - Jeff Duntemann

### Online Resources
- Ghidra tutorials (NSA)
- Radare2 documentation
- OWASP Reverse Engineering resources
- CTF (Capture The Flag) challenges
- HackTheBox machines

### Practice Platforms
- HackTheBox (realistic labs)
- TryHackMe (guided learning)
- PicoCTF (CTF challenges)
- OverTheWire wargames
- Crackmes.one (reverse engineering challenges)

---

## Common Patterns You'll Encounter

### Function Prologue
```asm
; Save old stack frame and return address
push rbp
mov rbp, rsp
sub rsp, 0x20        ; Allocate local variables
```

### Loop Pattern
```asm
mov ecx, 10          ; Counter
loop_start:
; Loop body
dec ecx
jnz loop_start        ; Jump if not zero
```

### Conditional Jump
```asm
cmp eax, ebx         ; Compare
je equal_case        ; Jump if equal
; Not equal code
jmp end
equal_case:
; Equal code
end:
```

### Switch Statement
```asm
; Often implemented as jump table
lea rax, [jump_table]
movzx ecx, byte ptr [rdi]
jmp qword ptr [rax + rcx*8]
```

---

## Summary

Reverse engineering is a powerful skill that combines:
- Binary format knowledge
- Assembly language understanding
- Debugging techniques
- System knowledge
- Problem-solving

**Key Takeaways:**
1. Learn binary formats (ELF, PE)
2. Understand assembly for your target architecture
3. Practice with debuggers and disassemblers
4. Analyze real programs (start simple)
5. Always consider legal and ethical implications

---

*This educational material is intended for learning and legitimate security research only.*
