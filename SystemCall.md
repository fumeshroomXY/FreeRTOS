# User Mode vs Kernel Mode
Modern operating systems (like Linux, Windows, or macOS) separate execution into **two privilege levels**:
| Mode            | Who runs here          | Access               |
| --------------- | ---------------------- | -------------------- |
| **User Mode**   | Normal programs (apps) | Limited access       |
| **Kernel Mode** | OS kernel              | Full hardware access |

## User Mode
Your application runs here.

Example programs:
- Browser
- Text editor
- Your C program

Restrictions:
- Cannot access hardware directly
- Cannot access other process memory
- Cannot execute privileged CPU instructions

This protects the system.

## Kernel Mode
The OS kernel runs here.

It can:
- Access hardware
- Manage memory
- Control processes
- Handle interrupts
- Schedule tasks

Because kernel mode has full control, a bug here can crash the whole system.

## Why User Mode Protection Exists
Without user/kernel separation:
```markdown
Bug in one program
        ↓
Overwrite OS memory
        ↓
Whole system crashes
```
This was common in early systems like MS-DOS.

Modern OS design prevents that.

# What is a System Call?
User programs still need OS services like:
- reading files
- allocating memory
- creating processes
- sending network packets

But user programs cannot directly access hardware.

So they use system calls.

A system call is a **controlled entry point from User Mode → Kernel Mode**.

```markdown
+-------------------------+
|        User Mode        |
|-------------------------|
|   Application Program   |
|   C library (glibc)     |
+-----------▲-------------+
            │ System Call
            ▼
+-------------------------+
|       Kernel Mode       |
|-------------------------|
|   Process scheduler     |
|   Memory manager        |
|   File system           |
|   Device drivers        |
+-------------------------+
```

Example:
```c
read(fd, buffer, 100);
```

Execution flow:
```markdown
User program(User Mode)
   │
   │ system call instruction
   ▼
Switch to Kernel Mode
   │
Kernel executes service
   │
Return instruction
   ▼
Back to User Mode
```

# Why Programs Crash
Crashes usually happen because the OS detects **illegal behavior in User Mode**.

Examples:
## 1. Invalid Memory Access
```c
int *p = NULL;
*p = 10;
```
Process tries to access memory **it does not own**.

CPU raises segmentation fault.

## 2. Illegal Instruction
If a user program executes a privileged instruction:
```markdown
disable interrupts
```
Only kernel mode can do that.

CPU generates exception → program terminated.

## 3. Stack Overflow
Infinite recursion:
```c
void f() { f(); }
```
Stack grows until memory protection stops it.


# Embedded Systems
In embedded systems without OS, everything runs in **privileged mode**.

So Bug in program -> Whole system crashes

RTOS (like FreeRTOS) often runs without strict user/kernel separation.
