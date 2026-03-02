In **FreeRTOS**, every task is represented internally by a **Task Control Block (TCB)**.

# TCB
When you create a task using:
```c
xTaskCreate()
```
FreeRTOS:
- Allocates memory for the **task stack**(stores CPU context + local variables)
- Allocates memory for the **TCB**
- Initializes the TCB fields
- Adds the task to the ready list

TCB is a complete object and `TaskHandle_t` is essentially a pointer to TCB.

## What Does the TCB Contain?
| Field               | Purpose                                                          |
| ------------------- | ---------------------------------------------------------------- |
| `pxTopOfStack`      | Pointer to current top of stack (critical for context switching) |
| `pxStack`           | Pointer to start of stack memory                                 |
| `uxPriority`        | Task priority                                                    |
| `xStateListItem`    | Used to place task in Ready/Blocked/Suspended lists              |
| `xEventListItem`    | Used when task waits for event (queue/semaphore/etc.)            |


## Memory Layout of Task Stack and TCB
Task stack and TCB are **NOT** guaranteed to be adjacent in memory.

```markdown
                +----------------------+
                |      TCB_t           |
                |----------------------|
                | pxTopOfStack --------|-----+
                | pxStack -------------|--+  |
                | uxPriority           |  |  |
                | state list item      |  |  |
                +----------------------+  |  |
                                          |  |
                                          |  |
                Stack Memory Region       |  |
                                          |  |
High Addr  +--------------------------+<--|--|
           |  R0                      |   |  
           |  R1                      |   |  
           |  R2                      |   |  
           |  R3                      |   |  
           |  R12                     |   |  
           |  LR                      |   |  
           |  PC                      |   |  
           |  xPSR                    |   |  
           |  R4–R11                  |   |  
           |  Local variables         |   |  
Low Addr   +--------------------------+<--|  
```

## Context Switch and Stack Layout
During a context switch:

**When task is switched OUT:**

- Registers are pushed onto stack
- `pxTopOfStack` is updated in TCB

**When task is switched IN:**

- `pxTopOfStack` is read from TCB
- Registers restored from stack
- Task resumes

So the TCB does **NOT store registers directly** —
it stores the **pointer to where they are saved on the stack**.
