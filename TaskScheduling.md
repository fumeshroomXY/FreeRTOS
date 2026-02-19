# Task Scheduling
The mechanism that decides which task runs at any given time.  

In FreeRTOS, the rule is simple:  
**The highest-priority Ready task always runs.**

## How FreeRTOS Implements It
FreeRTOS maintains:
- One Ready list per priority
- Tasks move between states:
  - Ready
  - Running
  - Blocked
  - Suspended

Example:
```markdown
Ready Lists
Priority 5 → [Task A]
Priority 4 → [Task B, Task C]
Priority 3 → [...]
...

The scheduler always selects:
Highest priority non-empty Ready list: Task A
```

# Preemptive Scheduling
A higher-priority task can **interrupt (preempt)** a currently running lower-priority task.
Enabled by:
```c
#define configUSE_PREEMPTION 1
```
A higher-priority task can preempt a lower-priority task immediately, **even in the middle of a tick period**.  
It does **NOT** have to wait for the next tick.

When:
- A higher-priority task becomes Ready
- An ISR unblocks a high-priority task
- A task calls vTaskDelay(), etc.

Then:
- Context of current task is saved
- CPU switches to higher-priority task

Example:
```markdown
Time →

Task B (Priority 2) running

Interrupt occurs
ISR gives semaphore to Task A (Priority 4)

Scheduler runs
Task A immediately preempts Task B
```

# Time-Slice Rotation
Applies only to tasks with the SAME priority

Enabled by:
```c
#define configUSE_TIME_SLICING 1
```
- Each tick interrupt rotates tasks in the same-priority Ready list.
- Each task runs for one tick period (typically 1 ms).

Example:
```markdown
Task A, B, C with the same priority 3

Tick 1 → A
Tick 2 → B
Tick 3 → C
Tick 4 → A
...
```

# `vTaskDelay()`
```c
void vTaskDelay( TickType_t xTicksToDelay );
```
Blocks the calling task for **a specified number of tick periods**.

During this time:
- The task state becomes **Blocked**
- CPU is given to another Ready task
- After the delay expires → task returns to **Ready** state(NOT starts running immediately)

Example:
```c
void Task(void *pvParameters)
{
    while (1)
    {
        doSomething();
        vTaskDelay(pdMS_TO_TICKS(100));
    }
}
```

## Delay Resolution Depends on Tick Rate
```c
#define configTICK_RATE_HZ 1000   // Tick period = 1 ms
#define configTICK_RATE_HZ 100    // Tick period = 10 ms

vTaskDelay(1);   // 1 ms (if 1000Hz) or 10 ms (if 100Hz)

// Always use:
pdMS_TO_TICKS(ms)
```

## How `vTaskDelay` Affects Scheduling
When you call it:
- Task → Blocked
- Scheduler immediately runs
- Lower-priority task can now execute

So it voluntarily gives up CPU.


# FreeRTOS Task Design Best Practices

## All tasks should either block or yield when they have nothing to do.
Blocking APIs include:
- vTaskDelay()
- vTaskDelayUntil()
- xQueueReceive() (with timeout)
- xSemaphoreTake()
- ulTaskNotifyTake()
- xEventGroupWaitBits()


These are preferred over busy wait.
```c
// ❌ Bad design
while(1)
{
    if(flag == 1)
        process();
}
```


## High-priority tasks are for emergency handling
High-priority tasks should be used for:
- Fault handling
- Emergency response
- Critical control loops
- Real-time events

Not for heavy background processing.

## Avoid using normal `delay()` or dead loops
Use RTOS blocking APIs like:
- vTaskDelay()
- xQueueReceive()
- xSemaphoreTake()

Because regular `delay()` blocks the whole CPU (bare-metal style).

## Do not use too many priority levels
Typically 3–5 levels are enough.

Too many priorities:

- Makes debugging harder
- Increases risk of priority inversion
- Makes system behavior unpredictable

## Never let high-priority tasks occupy CPU for a long time
High-priority tasks must:
- Run quickly
- Finish quickly
- Block quickly

Otherwise:
- Lower-priority tasks will **starve**
- System responsiveness suffers

