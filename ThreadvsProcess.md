# Process
A process is an **independent** running program.

Examples:
- Opening Google Chrome
- Running Microsoft Word
- Executing a compiled C program

Each process has its own resources:
- **Separate memory space**
- Own stack and heap
- Own file descriptors
- Own Process Control Block (PCB)

**Characteristics**
- Heavyweight
- Strong isolation
- Communication between processes requires **IPC(Inter-Process Communication)**: pipes, shared memory, message queues

# Thread
A thread is the **smallest unit** of execution inside a process.

A process can have **multiple threads**.

Example:

A browser like Google Chrome might use threads for:
- UI rendering thread
- Network download thread
- JavaScript execution thread


**Threads share process resources**

All threads share:
- Code
- Global variables
- Heap memory
- File handles

But each thread has its own:
- **Stack**
- Program Counter (PC)
- Registers

```markdown
Process
│
├── Thread 1
│   └── Stack
│
├── Thread 2
│   └── Stack
│
└── Thread 3
    └── Stack
```

# Process vs Thread
| Feature       | Process                                 | Thread                                   |
| ------------- | --------------------------------------- | ---------------------------------------- |
| Memory        | Separate memory space                   | Shared memory                            |
| Isolation     | Strong                                  | Weak                                     |
| Creation cost | High                                    | Low                                      |
| Communication | IPC required                            | Shared variables                         |
| Crash impact  | One process crash doesn't affect others | One thread crash may crash whole process |

# Thread in RTOS
In systems like **FreeRTOS**:
- There is usually no process concept.
- **Only tasks** (similar to threads).

Why?

Embedded systems often:
- Run a single application
- Share the same memory
- Avoid heavy process overhead.

So **FreeRTOS tasks ≈ threads**, not processes.


# Thread and Interrupt
## Thread vs. Interrupt
| Feature           | Thread        | Interrupt          |
| ----------------- | ------------- | ------------------ |
| Trigger           | Scheduler     | Hardware event     |
| Execution control | OS/RTOS       | Hardware           |
| Context           | Task context  | Interrupt context  |
| Stack             | Task stack    | ISR stack          |
| Priority          | Task priority | Interrupt priority |
| Blocking allowed  | Yes           | No                 |

**Interrupts cannot block.**

So functions like:
```c
vTaskDelay()
xQueueReceive()
```
cannot be used inside interrupts in FreeRTOS.

Instead you must use **FromISR APIs**.
```c
xQueueSendFromISR()
xSemaphoreGiveFromISR()
```

## Typical Interrupt + Thread Workflow
In embedded systems the interrupt does minimal work and wakes a thread.

UART receive example:
```markdown
UART hardware receives data
        |
        v
Interrupt occurs
        |
        v
ISR copies data to buffer
        |
        v
ISR notifies thread
        |
        v
Thread processes data
```

FreeRTOS Example: 
```c
// Task
void UART_Task(void *pvParameters)
{
    while(1)
    {
        ulTaskNotifyTake(pdTRUE, portMAX_DELAY);
        process_uart_data();
    }
}

// ISR
void UART_IRQHandler(void)
{
    BaseType_t xHigherPriorityTaskWoken = pdFALSE;

    vTaskNotifyGiveFromISR(uartTaskHandle, &xHigherPriorityTaskWoken);

    portYIELD_FROM_ISR(xHigherPriorityTaskWoken);
}
```

Flow:
```markdown
UART interrupt
     ↓
ISR runs
     ↓
Notify task
     ↓
Scheduler runs task
```

## Important Concept: Preemption
Interrupts have **higher priority than threads**.

So they can interrupt a running thread immediately.

Example:
```markdown
Thread A running
       |
       | timer interrupt
       v
ISR executes
       |
       v
Scheduler may switch to higher-priority thread
```
