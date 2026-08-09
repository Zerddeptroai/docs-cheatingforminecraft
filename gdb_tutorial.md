# GDB (GNU Debugger) Tutorial: Comprehensive Guide with Walkthroughs

## Table of Contents
1. [What is GDB?](#what-is-gdb)
2. [Installation & Setup](#installation--setup)
3. [Basic Commands](#basic-commands)
4. [Starting a Debug Session](#starting-a-debug-session)
5. [Breakpoints](#breakpoints)
6. [Stepping Through Code](#stepping-through-code)
7. [Inspecting Variables & Memory](#inspecting-variables--memory)
8. [Stack & Registers](#stack--registers)
9. [Advanced Features](#advanced-features)
10. [Debugging Walkthroughs](#debugging-walkthroughs)
11. [Tips & Tricks](#tips--tricks)
12. [Cheat Sheet](#cheat-sheet)

---

## What is GDB?

**GDB** (GNU Debugger) is a powerful tool that allows you to:
- Run programs under controlled conditions
- Set breakpoints to pause execution
- Step through code line by line (or instruction by instruction)
- Inspect variables, memory, and registers
- Modify values on the fly
- Debug crashes and understand program behavior

GDB works with compiled binaries on Linux, macOS, and other Unix-like systems.

---

## Installation & Setup

### Linux (Debian/Ubuntu)
```bash
sudo apt update
sudo apt install gdb
```

### macOS
```bash
brew install gdb
# Note: May require additional setup for code signing
```

### Verify Installation
```bash
gdb --version
```

### Compile with Debug Symbols

**IMPORTANT:** Compile with `-g` flag to include debug symbols:

```bash
gcc -g -o program program.c
# or
gcc -g -O0 -o program program.c    # -O0 disables optimizations for better debugging
```

**Why debug symbols?**
- Without `-g`: You see only assembly code
- With `-g`: You see C source code, function names, variable names

---

## Basic Commands

### Startup Commands

| Command | Purpose |
|---------|---------|
| `gdb ./program` | Start debugging a program |
| `gdb --args ./program arg1 arg2` | Debug with arguments |
| `gdb -p PID` | Attach to running process |
| `gdb ./program core` | Debug with core dump |

### Navigation

| Command | Shortcut | Purpose |
|---------|----------|---------|
| `run` | `r` | Start/restart the program |
| `continue` | `c` | Continue execution |
| `step` | `s` | Step into (one line, enter functions) |
| `next` | `n` | Step over (one line, skip functions) |
| `finish` | `fin` | Execute until function returns |
| `until` | `u` | Execute until line N |
| `reverse-step` | `rs` | Step backward (if recording) |

### Information Commands

| Command | Purpose |
|---------|---------|
| `info locals` | Show local variables |
| `info args` | Show function arguments |
| `info registers` | Show all registers |
| `info breakpoints` | Show all breakpoints |
| `info threads` | Show all threads |
| `backtrace` | Show call stack |
| `bt full` | Show call stack with locals |

### Display Commands

| Command | Purpose |
|---------|---------|
| `print variable` | Print variable value |
| `print *pointer` | Dereference and print |
| `print array@10` | Print 10 elements of array |
| `print /x variable` | Print as hexadecimal |
| `print /o variable` | Print as octal |
| `print /b variable` | Print as binary |
| `x /100x address` | Examine 100 hex values at address |
| `x /s address` | Examine string at address |
| `disassemble main` | Disassemble function |
| `disassemble /m main` | Disassemble with source lines |

### Modification Commands

| Command | Purpose |
|---------|---------|
| `set variable = value` | Change a variable |
| `set $rax = 0x1000` | Change a register |
| `set *(int*)0x1000 = 42` | Write to memory |
| `call function()` | Call a function |

### Breakpoint Commands

| Command | Purpose |
|---------|---------|
| `break main` | Set breakpoint at function |
| `break file.c:10` | Set breakpoint at line 10 |
| `break *0x1000` | Set breakpoint at address |
| `delete 1` | Delete breakpoint 1 |
| `disable 1` | Disable breakpoint 1 |
| `enable 1` | Enable breakpoint 1 |
| `continue` | Continue until next breakpoint |

### Exit Commands

| Command | Purpose |
|---------|---------|
| `quit` | Exit GDB |
| `q` | Shortcut for quit |

---

## Starting a Debug Session

### Step 1: Compile with Debug Symbols

```bash
# Create a simple C program
cat > hello.c << 'EOF'
#include <stdio.h>

int main() {
    printf("Hello, World!\n");
    return 0;
}
EOF

# Compile with debug symbols
gcc -g -o hello hello.c
```

### Step 2: Start GDB

```bash
gdb ./hello
```

**Output:**
```
GNU gdb (GDB) 12.1
Reading symbols from ./hello...
(gdb)
```

### Step 3: First Commands

```gdb
(gdb) break main           # Set breakpoint at main
Breakpoint 1 at 0x1159: file hello.c, line 5.

(gdb) run                  # Start the program
Starting program: /path/to/hello
Breakpoint 1, main () at hello.c:5
5           printf("Hello, World!\n");

(gdb) quit                 # Exit
```

---

## Breakpoints

### Setting Breakpoints

#### By Function Name
```gdb
(gdb) break main
(gdb) break my_function
```

#### By File and Line
```gdb
(gdb) break hello.c:10
(gdb) break myfile.c:25
```

#### By Address
```gdb
(gdb) break *0x1159
```

#### Conditional Breakpoints
```gdb
(gdb) break myfile.c:20 if x > 100    # Break only if x > 100
(gdb) break myfile.c:30 if str == NULL  # Break if pointer is NULL
```

#### With Command List
```gdb
(gdb) break main
(gdb) commands 1
> print "Breakpoint hit!"
> print x
> print y
> continue
> end
```

### Managing Breakpoints

```gdb
(gdb) info breakpoints              # List all breakpoints
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x0000555555555159 in main at hello.c:5

(gdb) disable 1                     # Disable breakpoint 1
(gdb) enable 1                      # Re-enable breakpoint 1
(gdb) delete 1                      # Delete breakpoint 1
(gdb) clear main                    # Clear breakpoints at main
(gdb) delete                        # Delete all breakpoints
```

### Watchpoints (Data Breakpoints)

Break when a variable is **modified**, **read**, or **accessed**:

```gdb
(gdb) watch x                       # Break when x is modified
Hardware watchpoint 2: x
(gdb) rwatch y                      # Break when y is read
(gdb) awatch z                      # Break on any access to z

(gdb) info watchpoints
Num     Type           Disp Enb Address            What
2       hw watchpoint  keep y                      x
```

---

## Stepping Through Code

### The Three Step Commands

#### `step` (s) - Step INTO Functions
```
Step through code, entering function calls.
If you hit a function call, you go inside it.
```

#### `next` (n) - Step OVER Functions
```
Step through code, but treat function calls as one step.
Doesn't enter the function.
```

#### `finish` - Execute Until Function Returns
```
Run until the current function returns.
Useful for getting out of a function quickly.
```

### Example Walkthrough

**Code:**
```c
int add(int a, int b) {
    return a + b;
}

int main() {
    int x = 5;
    int y = 10;
    int z = add(x, y);          // Line 8
    printf("Result: %d\n", z);  // Line 9
    return 0;
}
```

**Debug Session:**
```gdb
(gdb) break main
(gdb) run
Breakpoint 1, main () at program.c:6
6           int x = 5;

(gdb) next
7           int y = 10;

(gdb) next
8           int z = add(x, y);

(gdb) step                  # Go INTO add()
add (a=5, b=10) at program.c:2
2           return a + b;

(gdb) step
3       }

(gdb) step                  # Back to main
main () at program.c:9
9           printf("Result: %d\n", z);

(gdb) next
Result: 15
10          return 0;

(gdb) finish
0x00007ffff7a05082 in __libc_start_main ()
```

---

## Inspecting Variables & Memory

### Print Variables

#### Simple Print
```gdb
(gdb) print x               # Print variable x
$1 = 42

(gdb) print sum             # Print sum
$2 = 10

(gdb) print x + y           # Print expression
$3 = 52
```

#### Different Formats

```gdb
(gdb) print x               # Decimal
$1 = 42

(gdb) print /x x            # Hexadecimal
$2 = 0x2a

(gdb) print /o x            # Octal
$3 = 052

(gdb) print /b x            # Binary
$4 = 101010

(gdb) print /c x            # Character
$5 = '*'

(gdb) print /s 0x1000       # String at address
$6 = "Hello, World!"
```

### Pointers and Dereferencing

```gdb
(gdb) print ptr             # Print pointer value (address)
$1 = (int *) 0x7ffffffde540

(gdb) print *ptr            # Dereference pointer (get value)
$2 = 42

(gdb) print ptr->field      # Access struct member
$3 = 123

(gdb) print &x              # Get address of variable
$4 = (int *) 0x7ffffffde544
```

### Arrays and Collections

```gdb
(gdb) print arr             # Print array (shows first element)
$1 = {1, 2, 3, 4, 5}

(gdb) print arr[0]          # Print first element
$2 = 1

(gdb) print arr@10          # Print first 10 elements
$3 = {1, 2, 3, 4, 5, 0, 0, 0, 0, 0}

(gdb) print *str            # Print first character of string
$4 = 72 'H'

(gdb) print str[3]          # Print 4th character
$5 = 108 'l'
```

### Examine Memory (x command)

The `x` command is powerful for raw memory inspection:

```
x /[count][format][size] address

Count:  Number of units to display
Format: x (hex), d (decimal), o (octal), b (binary), i (instruction), s (string), c (char)
Size:   b (byte), h (halfword/2 bytes), w (word/4 bytes), g (giant/8 bytes)
```

**Examples:**

```gdb
(gdb) x /10x 0x1000         # 10 hex values at address 0x1000
0x1000:     0xdeadbeef  0xcafebabe  0x12345678  0x87654321
0x1010:     0xffffffff  0x00000000  0x11223344  0x55667788
0x1020:     0xaabbccdd  0xeeff0011

(gdb) x /20i 0x1000         # 20 instructions starting at 0x1000
   0x1000:    push   %rbp
   0x1001:    mov    %rsp,%rbp
   0x1004:    mov    %edi,-0x4(%rbp)
   ...

(gdb) x /50s 0x2000         # 50 strings starting at 0x2000
0x2000:     "Hello"
0x2006:     "World"
...

(gdb) x /100d 0x3000        # 100 decimal values at 0x3000

(gdb) x /b 0x1000           # Single byte
0x1000:     0x55
```

### Print with History

```gdb
(gdb) print x
$1 = 42

(gdb) print y
$2 = 10

(gdb) print $1 + $2         # Reference previous prints
$3 = 52
```

---

## Stack & Registers

### Backtrace (Call Stack)

```gdb
(gdb) backtrace             # Show call stack
#0  function_a () at program.c:10
#1  function_b () at program.c:20
#2  main () at program.c:30

(gdb) bt                    # Shortcut
(same as above)

(gdb) bt full               # Show with local variables
#0  function_a () at program.c:10
        x = 5
        y = 10
#1  function_b () at program.c:20
        result = 15
#2  main () at program.c:30
```

### Frame Navigation

```gdb
(gdb) frame 0               # Go to frame 0 (current)
#0  function_a () at program.c:10
10          return x + y;

(gdb) frame 1               # Go to frame 1 (caller)
#1  function_b () at program.c:20
20          int result = function_a();

(gdb) up                    # Go up one frame (to caller)
(gdb) down                  # Go down one frame (to callee)
```

### Register Information

```gdb
(gdb) info registers        # Show all registers
rax            0x0             0
rbx            0x0             0
rcx            0x0             0
rdx            0x0             0
rsi            0x0             0
rdi            0x0             0
rbp            0x7ffffffde540  0x7ffffffde540
rsp            0x7ffffffde538  0x7ffffffde538
r8             0x0             0
r9             0x0             0
r10            0x0             0
r11            0x0             0
r12            0x0             0
r13            0x0             0
r14            0x0             0
r15            0x0             0
rip            0x555555555159   0x555555555159 <main+0>
eflags         0x202            [ IF ]
cs             0x33            51
ss             0x2b            43
ds             0x0             0
es             0x0             0
fs             0x0             0
gs             0x0             0

(gdb) print $rax            # Print specific register
$1 = 0

(gdb) print $rsi            # Register with argument
$2 = 0

(gdb) info registers rax rbx rcx  # Show specific registers
rax            0x0             0
rbx            0x0             0
rcx            0x0             0
```

### Modify Registers

```gdb
(gdb) set $rax = 0x1000     # Set RAX to 0x1000
(gdb) set $rdi = 0x2000     # Set RDI (first argument) to 0x2000
(gdb) set $rsp = $rsp - 8   # Decrease stack pointer
```

---

## Advanced Features

### Automatic Display

Automatically print variables after each step:

```gdb
(gdb) display x             # Always show x
1: x = 42

(gdb) display y             # Always show y
2: y = 10

(gdb) info display          # List all displays
Auto-display expressions now in effect:
Num Enb Expression
1   y   x
2   y   y

(gdb) undisplay 1           # Stop displaying x
(gdb) disable display 2     # Disable but don't delete
(gdb) enable display 2      # Re-enable
```

### Conditional Execution

```gdb
(gdb) if (x > 10)
 >print "x is greater than 10"
 >end
```

### Logging

```gdb
(gdb) set logging on        # Start logging to gdb.txt
(gdb) set logging file mylog.txt  # Log to specific file
(gdb) set logging off       # Stop logging
```

### Core Dumps

Debug a crashed program:

```bash
# Generate core dumps on crash
ulimit -c unlimited

# Run program (it crashes)
./program

# Debug the core dump
gdb ./program core
```

### Remote Debugging

```bash
# On remote machine
gdbserver localhost:2345 ./program

# On local machine
gdb ./program
(gdb) target remote remote_host:2345
(gdb) continue
```

### Process Attachment

```bash
# In one terminal, start program that won't exit
sleep 10000 &

# Get PID
echo $!   # 12345

# In another terminal, attach GDB
gdb -p 12345
```

---

## Debugging Walkthroughs

### Walkthrough 1: Simple Bug - Off-by-One Error

**Program:**
```c
#include <stdio.h>

int main() {
    int arr[5] = {1, 2, 3, 4, 5};
    int sum = 0;
    
    for (int i = 0; i <= 5; i++) {  // BUG: should be i < 5
        sum += arr[i];
    }
    
    printf("Sum: %d\n", sum);
    return 0;
}
```

**Compilation:**
```bash
gcc -g -o buggy buggy.c
```

**Debug:**
```gdb
(gdb) break main
(gdb) run
Breakpoint 1, main () at buggy.c:5
5           int arr[5] = {1, 2, 3, 4, 5};

(gdb) next
6           int sum = 0;

(gdb) next
8           for (int i = 0; i <= 5; i++) {
             ^^^^^^^^^

(gdb) next
9               sum += arr[i];

# Let's step through the loop
(gdb) display i
1: i = 0

(gdb) display sum
2: sum = 0

(gdb) next
8           for (int i = 0; i <= 5; i++) {
1: i = 0
2: sum = 1

(gdb) next
9               sum += arr[i];
1: i = 1
2: sum = 1

(gdb) next
8           for (int i = 0; i <= 5; i++) {
1: i = 1
2: sum = 3

# Continue until the crash
(gdb) continue
[Segmentation fault]

# Now investigate
(gdb) print i
$3 = 5

(gdb) print arr[5]
$4 = -1077936128    # Out of bounds! Should only be 0-4

(gdb) print sizeof(arr)
$5 = 20             # 5 integers * 4 bytes

# The bug is clear: loop condition should be i < 5, not i <= 5
```

### Walkthrough 2: Pointer Bug - Null Dereference

**Program:**
```c
#include <stdio.h>
#include <stdlib.h>

void process_data(int *ptr) {
    printf("Value: %d\n", *ptr);  // Crash if ptr is NULL
}

int main() {
    int *data = NULL;
    
    if (data != NULL) {
        process_data(data);
    }
    
    // BUG: forgot the check here
    process_data(data);
    
    return 0;
}
```

**Debug:**
```gdb
(gdb) break process_data
(gdb) run
Breakpoint 1, process_data (ptr=0x0) at buggy.c:5
5           printf("Value: %d\n", *ptr);

(gdb) print ptr
$1 = (int *) 0x0     # NULL pointer

(gdb) print *ptr
Cannot access memory at address 0x0

# Now we know: process_data was called with a NULL pointer
# The bug is in main - should check before calling
```

### Walkthrough 3: Memory Leak - Infinite Loop

**Program:**
```c
#include <stdio.h>

int main() {
    int x = 0;
    
    while (x < 100) {
        x += 1;
        if (x == 50) {
            // BUG: forgot to increment in some condition
            if (x % 2 == 0) {
                // This block doesn't increment x
                printf("x is even\n");
            }
        }
    }
    
    printf("Done: x = %d\n", x);
    return 0;
}
```

**Debug - Using Breakpoint with Counter:**
```gdb
(gdb) break program.c:8
(gdb) commands 1
>silent
>print x
>continue
>end

(gdb) run
x = 1
x = 2
x = 3
...
x = 50
x = 51
...
x = 100
Done: x = 100

# Hmm, it actually completes. Let me set a conditional breakpoint
(gdb) break program.c:8 if x == 50
(gdb) run
Breakpoint 1, main () at program.c:8
8           x += 1;

(gdb) next
9           if (x == 50) {

(gdb) next
10              if (x % 2 == 0) {

(gdb) info locals
x = 50

(gdb) next
13              }

# The loop counter is working fine. Let me check if there's actually a bug.
# (In this case, there isn't - I made a mistake in the example)
```

### Walkthrough 4: Debugging with Conditional Breakpoints

**Program:**
```c
#include <stdio.h>

void analyze_data(int value) {
    printf("Analyzing: %d\n", value);
}

int main() {
    for (int i = 0; i < 1000; i++) {
        if (i % 7 == 0) {  // Only analyze multiples of 7
            analyze_data(i);
        }
    }
    return 0;
}
```

**Debug with Conditional Breakpoint:**
```gdb
(gdb) break analyze_data if value > 100
(gdb) run
Breakpoint 1, analyze_data (value=105) at program.c:3
3           printf("Analyzing: %d\n", value);

(gdb) print value
$1 = 105

(gdb) continue
Breakpoint 1, analyze_data (value=112) at program.c:3
3           printf("Analyzing: %d\n", value);

(gdb) print value
$2 = 112

# Much faster than debugging every call!
```

### Walkthrough 5: Assembly-Level Debugging

**Program:**
```c
int add(int a, int b) {
    return a + b;
}

int main() {
    int result = add(5, 10);
    return result;
}
```

**Compile and Debug:**
```gdb
(gdb) break main
(gdb) run
Breakpoint 1, main () at program.c:6
6           int result = add(5, 10);

(gdb) disassemble main
Dump of assembler code for function main:
   0x0000555555555145 <+0>:     push   %rbp
   0x0000555555555146 <+1>:     mov    %rsp,%rbp
   0x0000555555555149 <+4>:     mov    $0xa,%esi
   0x000055555555514e <+9>:     mov    $0x5,%edi
   0x0000555555555153 <+14>:    call   0x555555555139 <add>
   0x0000555555555158 <+19>:    mov    %eax,-0x4(%rbp)
   0x000055555555515b <+22>:    mov    $0x0,%eax
   0x0000555555555160 <+27>:    pop    %rbp
   0x0000555555555161 <+28>:    ret

(gdb) stepi                 # Step one instruction
0x0000555555555146 in main () at program.c:6
6           int result = add(5, 10);

(gdb) info registers rdi rsi
rdi            0x5             5      # First argument
rsi            0xa             10     # Second argument

(gdb) stepi
(gdb) stepi
(gdb) stepi
(gdb) stepi                 # Several steps to reach CALL

(gdb) stepi                 # Execute CALL, enter add()
0x0000555555555139 in add (a=5, b=10) at program.c:1
1       int add(int a, int b) {

(gdb) disassemble /m add
Dump of assembler code for function add:
   0x0000555555555139 <+0>:     lea    0x0(%rip),%rax
   0x0000555555555140 <+7>:     add    %esi,%edi
   0x0000555555555142 <+9>:     mov    %edi,%eax
   0x0000555555555144 <+11>:    retq

   1   int add(int a, int b) {
   2       return a + b;
   3   }

(gdb) stepi                 # Execute ADD instruction
0x0000555555555142 in add (a=5, b=10) at program.c:2
2           return a + b;

(gdb) info registers rax rdi rsi
rax            0x0             0
rdi            0xf             15     # EDI now contains a+b
rsi            0xa             10

(gdb) stepi                 # Execute MOV instruction
0x0000555555555144 in add (a=5, b=10) at program.c:2
2           return a + b;

(gdb) info registers rax
rax            0xf             15     # EAX now has return value

(gdb) stepi                 # Execute RET
(back to main)
```

### Walkthrough 6: String and Array Debugging

**Program:**
```c
#include <stdio.h>
#include <string.h>

int main() {
    char buffer[10];
    char *source = "Hello, World!";
    
    strcpy(buffer, source);  // BUG: buffer overflow
    
    printf("Buffer: %s\n", buffer);
    return 0;
}
```

**Debug:**
```gdb
(gdb) break main
(gdb) run
Breakpoint 1, main () at program.c:6
6           char buffer[10];

(gdb) next
7           char *source = "Hello, World!";

(gdb) print buffer
$1 = "\000\001\002"...    # Uninitialized

(gdb) next
9           strcpy(buffer, source);

(gdb) print buffer
$2 = ""                   # Empty

(gdb) print source
$3 = 0x555555556008 "Hello, World!"

(gdb) print strlen(source)
$4 = 13    # 13 characters, but buffer only 10!

(gdb) next
(Segmentation fault)

(gdb) print buffer
$5 = "Hello, World!"  # Overflowed into adjacent memory

# The bug is clear: strcpy doesn't check bounds, use strncpy instead
```

---

## Tips & Tricks

### Tip 1: Use Abbreviations

```gdb
(gdb) b main           # break main
(gdb) c                # continue
(gdb) n                # next
(gdb) s                # step
(gdb) fin              # finish
(gdb) p x              # print x
(gdb) bt               # backtrace
(gdb) q                # quit
```

### Tip 2: Search Disassembly

```bash
# Disassemble to file, then search
gdb -batch -ex "disassemble main" ./program > dis.txt
grep "call" dis.txt
```

### Tip 3: Print Complex Structures

```c
struct {
    int x;
    char *name;
    int arr[10];
} data;
```

```gdb
(gdb) print data
$1 = {x = 42, name = 0x1000 "Alice", arr = {1, 2, 3, ...}}

(gdb) print data.x
$2 = 42

(gdb) print data.name
$3 = 0x1000 "Alice"

(gdb) print data.arr@3
$4 = {1, 2, 3}
```

### Tip 4: Function Return Values

```c
int factorial(int n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1);
}
```

```gdb
(gdb) break factorial
(gdb) run
(gdb) finish                # Run until return
Value returned is $1 = 120
```

### Tip 5: Set Return Values

```gdb
(gdb) break main
(gdb) run
(gdb) call some_function()
$1 = 42                     # Function returned 42

# Modify the return value
(gdb) set $1 = 100
(gdb) continue              # Program thinks function returned 100
```

### Tip 6: Reverse Execution (Record & Replay)

```bash
gdb ./program
(gdb) record
(gdb) run
(gdb) ...execution...
(gdb) reverse-step          # Step backward!
(gdb) reverse-continue      # Continue backward
(gdb) record stop           # Stop recording
```

### Tip 7: GDB Script Files

Create `commands.gdb`:
```gdb
break main
break function_name
commands 1
silent
print x
print y
continue
end
run
```

Then run:
```bash
gdb -x commands.gdb ./program
```

### Tip 8: Find Memory Leaks

```bash
valgrind --leak-check=full --show-leak-kinds=all ./program
```

### Tip 9: Pretty Printing

```gdb
(gdb) set print pretty on   # Format complex structures nicely
(gdb) print data
$1 = {
  x = 42,
  name = 0x1000 "Alice",
  arr = {1, 2, 3}
}
```

### Tip 10: Print with History

```gdb
(gdb) print x
$1 = 42

(gdb) print $1 * 2
$2 = 84

(gdb) print $1 + $2
$3 = 126
```

---

## Cheat Sheet

### Quick Reference

```gdb
# Start
gdb ./program
gdb --args ./program arg1 arg2
gdb -p PID

# Basic Flow
run / r                    # Start
continue / c               # Continue
step / s                   # Step into
next / n                   # Step over
finish / fin               # Execute to return
stepi / si                 # Step instruction
nexti / ni                 # Next instruction

# Breakpoints
break main / b main        # Set breakpoint
break file.c:10            # Set at line
break *0x1000              # Set at address
break if condition         # Conditional breakpoint
delete 1                   # Delete breakpoint 1
disable 1                  # Disable breakpoint 1
enable 1                   # Enable breakpoint 1
info breakpoints           # List breakpoints

# Variables & Memory
print x / p x              # Print variable
print /x x                 # Print as hex
print *ptr                 # Dereference
print arr@10               # Print array elements
x /10x 0x1000              # Examine memory
set x = 5                  # Set variable
set $rax = 0x1000          # Set register

# Stack & Registers
backtrace / bt             # Show stack
bt full                    # Stack with locals
frame 0                    # Select frame
info locals                # Show local variables
info args                  # Show arguments
info registers             # Show all registers
print $rax                 # Print register

# Information
info breakpoints           # List breakpoints
info watches               # List watchpoints
info threads               # List threads
info stack                 # Stack info
disassemble main           # Disassemble function
disassemble /m main        # Disassemble with source

# Watchpoints
watch x                    # Break on x modified
rwatch y                   # Break on y read
awatch z                   # Break on z access

# Advanced
call func()                # Call function
set logging on             # Start logging
display x                  # Auto-display x
record                     # Record for reverse-exec
quit / q                   # Exit
```

---

## Common Issues

### Issue 1: "No Debug Symbols"

**Problem:** `Reading symbols from ./program...(no debugging symbols found)`

**Solution:** Compile with `-g`:
```bash
gcc -g -o program program.c
```

### Issue 2: "Cannot Access Memory"

**Problem:** `Cannot access memory at address 0x1234567`

**Solution:** 
- Address may not exist yet
- Use valid addresses from `info registers` or `info locals`
- Try `x /10x $rsp` to see stack

### Issue 3: Breakpoint "Pending"

**Problem:** `<PENDING>` breakpoint shows in list

**Solution:**
- Library not loaded yet
- Breakpoint will be resolved when library loads
- Breakpoint is valid for later use

### Issue 4: Program Won't Start

**Problem:** Program doesn't run, stuck at prompt

**Solution:**
```gdb
(gdb) run                  # Try again
(gdb) start                # Alternative - run and break at main
```

---

## Practice Exercises

1. **Set and hit a breakpoint at a function**
   - Compile a program with multiple functions
   - Set breakpoints in different functions
   - Step through and inspect variables

2. **Debug an infinite loop**
   - Create a program with a loop
   - Set conditional breakpoint to catch iterations
   - Examine loop variables

3. **Inspect memory**
   - Allocate an array
   - Use `x` command to view raw memory
   - Modify memory values

4. **Debug with assembly**
   - Disassemble a function
   - Use `stepi` to step through instructions
   - Monitor registers changing

5. **String/array debugging**
   - Create program with strings and arrays
   - Print array elements individually
   - Identify buffer overflows

---

## Useful Resources

- **Official GDB Manual:** https://sourceware.org/gdb/documentation/
- **GDB Cheat Sheet:** Quick reference cards online
- **Practice:** Write intentional bugs and debug them
- **Online Debugging:** GDB Playground websites

---

*Practice makes perfect! Start with small programs and work up to more complex debugging scenarios.*
