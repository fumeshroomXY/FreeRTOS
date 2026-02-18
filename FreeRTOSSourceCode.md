# FreeRTOS Source Code Structure Overview
## Top-Level Structure
```markdown
FreeRTOS/
│
├── FreeRTOS-Kernel/
│   ├── include/
│   ├── portable/
│   ├── tasks.c
│   ├── queue.c
│   ├── list.c
│   ├── timers.c
│   ├── event_groups.c
│   ├── stream_buffer.c
│   └── croutine.c
│
├── FreeRTOS-Plus/
├── Demo/
└── License/
```

### 1. Core Kernel Files (Hardware Independent)
Located inside:
```bash
FreeRTOS-Kernel/
```
Important core files:
| File              | Purpose                                     |
| ----------------- | ------------------------------------------- |
| `tasks.c`         | Task management (create, delete, scheduler) |
| `queue.c`         | Queues & semaphores                         |
| `list.c`          | Internal linked list implementation         |
| `timers.c`        | Software timers                             |
| `event_groups.c`  | Event groups                                |
| `stream_buffer.c` | Stream & message buffers                    |
| `croutine.c`      | Co-routines (older feature)                 |

These files are **portable and architecture independent**.

### 2. include/ (Public API Headers)
```bash
FreeRTOS-Kernel/include/
```

### 3. portable/ (Hardware-Specific Layer)
```bash
FreeRTOS-Kernel/portable/
```
This is the most important concept.
FreeRTOS separates:
```bash
Kernel logic  ← independent
Port layer    ← CPU/compiler dependent
```
Inside `portable/`:
```markdown
portable/
├── GCC/
│   ├── ARM_CM4F/
│   ├── ARM_CM3/
│   ├── RISC-V/
│   └── ...
├── IAR/
├── Keil/
└── MemMang/
```

This is why FreeRTOS can run on: ARM Cortex-M, RISC-V, MSP430 etc.  
Only the port layer changes.
#### Port Layer Contains
Each CPU port includes:
```bash
port.c
portmacro.h
```
Example: If you use Cortex-M4 with GCC:
```bash
portable/GCC/ARM_CM4F/
```

### 4. Memory Management (Heap_x.c)
Located inside:
```bash
portable/MemMang/
```
You choose ONE of these:
| File       | Description                          |
| ---------- | ------------------------------------ |
| `heap_1.c` | Simplest (no free)                   |
| `heap_2.c` | Free allowed, no coalescing          |
| `heap_3.c` | Wraps malloc/free                    |
| `heap_4.c` | Coalescing free blocks (**most common**) |
| `heap_5.c` | Multiple memory regions              |


# How It Fits in an Embedded Project
```markdown
Project/
│
├── main.c
├── startup_xxx.s
├── system_xxx.c
│
├── FreeRTOS-Kernel/
    ├── source/
    │   ├── include/
    │   ├── tasks.c
    │   ├── queue.c
    │   ├── ...
    ├── portable/
        ├── GCC/ARM_CM4F/
            ├── port.c
            └── portmacro.h
        └── MemMang/heap_4.c
    └── config/
        └── FreeRTOSConfig.h


```
