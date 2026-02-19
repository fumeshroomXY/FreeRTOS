In FreeRTOS, **SysTick** is a hardware timer used to generate the periodic interrupt that drives the RTOS scheduler.

It generates **an interrupt** at a fixed time interval (for example, every 1 ms).  
In FreeRTOS, this interrupt is called the **RTOS tick**.

## What does SysTick do in FreeRTOS?
SysTick is responsible for:
- Incrementing the **tick count**
- Managing **task delays** (`vTaskDelay`)
- Handling timeouts
- Triggering **context switching** (if preemption is enabled)
- Driving the scheduler timing

Every time the SysTick interrupt occurs:
- The tick counter increases.
- FreeRTOS checks if any blocked tasks should be unblocked.
- If a higher-priority task becomes ready, a context switch happens.

**Example:**  
If `configTICK_RATE_HZ = 1000`,
SysTick generates an interrupt every:
```c
// FreeRTOSConfig.h
#define configTICK_RATE_HZ    1000
1 / 1000 = 1 ms
```
So the RTOS tick period is **1 millisecond**.   
The RTOS tick interrupt happens 1000 times per second (1 ms period).

## How SysTick works internally
On Cortex-M:
- SysTick is a 24-bit down counter.
- It reloads automatically after reaching zero.
- It generates the **`SysTick_Handler`** interrupt.
