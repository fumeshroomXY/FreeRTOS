# A classic race condition problem in FreeRTOS

## Problem: Two Tasks Competing for a Shared Variable
```c
#include "FreeRTOS.h"
#include "task.h"

volatile uint32_t sharedCounter = 0;

void Task1(void *pvParameters)
{
    while(1)
    {
        sharedCounter++;   // NOT safe!
        vTaskDelay(pdMS_TO_TICKS(1));
    }
}

void Task2(void *pvParameters)
{
    while(1)
    {
        sharedCounter++;   // NOT safe!
        vTaskDelay(pdMS_TO_TICKS(1));
    }
}
```
### Why This Is a Problem
```c
sharedCounter++;
```
the incrementing operation is **NOT atomic**. It actually becomes something like this at CPU level:
- Load sharedCounter into register
- Add 1
- Store back to memory

Now imagine this timeline:
| Time | Task1     | Task2     |
| ---- | --------- | --------- |
| t1   | Load 0    |           |
| t2   |           | Load 0    |
| t3   | Add 1 → 1 |           |
| t4   |           | Add 1 → 1 |
| t5   | Store 1   |           |
| t6   |           | Store 1   |


Final result = **1** instead of **2**

This happens because:
- Preemptive scheduling allows context switch anytime
- Both tasks access the same variable
- Operation is not atomic

## Solution: Use Critical Section
```c
taskENTER_CRITICAL();
taskEXIT_CRITICAL();
```
- **Disable interrupts**
- **Prevent context switch**
- **Make the code inside atomic**

```c
void Task1(void *pvParameters)
{
    while(1)
    {
        taskENTER_CRITICAL();
        sharedCounter++;
        taskEXIT_CRITICAL();

        vTaskDelay(pdMS_TO_TICKS(1));
    }
}

void Task2(void *pvParameters)
{
    while(1)
    {
        taskENTER_CRITICAL();
        sharedCounter++;
        taskEXIT_CRITICAL();

        vTaskDelay(pdMS_TO_TICKS(1));
    }
}
```

## Important Notes
### 1. Keep Critical Sections SHORT
❌ Bad:
```c
taskENTER_CRITICAL();
printf("Hello");   // slow!
vTaskDelay(1000);  // NEVER DO THIS
taskEXIT_CRITICAL();
```
Blocking or long operations inside critical section = system freeze.

### 2. Use mutex when protecting larger shared resources
