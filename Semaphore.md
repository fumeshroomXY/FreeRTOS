# Semaphore
Signal or control access — **no data transfer**

A door key — not a mailbox.

## Binary Semaphore
A binary semaphore can only have two states:
- Available (1)
- Unavailable (0)

### `xSemaphoreCreateBinary()`
Creates a binary semaphore.
```c
SemaphoreHandle_t xSemaphoreCreateBinary(void);
```
- When created, the semaphore is empty (unavailable).
- You **must `xSemaphoreGive()` it first** before it can be taken.

Example:
```c
SemaphoreHandle_t xBinarySemaphore;

void init(void)
{
    xBinarySemaphore = xSemaphoreCreateBinary();
}
```

### `xSemaphoreTake()`
Wait (block) until the semaphore becomes available.
```c
BaseType_t xSemaphoreTake(
    SemaphoreHandle_t xSemaphore,
    TickType_t xTicksToWait
);
```
`xTicksToWait` controls how long the task is willing to wait if the semaphore is not available.
- `0` → don’t block, return `pdFALSE` if unavailable, may lead to time polling.
- `portMAX_DELAY` → wait forever, task enters Blocked state
- `pdMS_TO_TICKS(100)` → wait 100ms

Example:
```c
if (xSemaphoreTake(xBinarySemaphore, portMAX_DELAY) == pdTRUE)
{
    // Semaphore received
}
```
If semaphore is not available:
- Task goes to **Blocked state**
- Scheduler runs another task

### `xSemaphoreGive()`
Release (signal) the semaphore.
```c
BaseType_t xSemaphoreGive(SemaphoreHandle_t xSemaphore);
```
- Semaphore becomes available (if not already)
- Highest priority waiting task is unblocked and may cause immediate context switch

### Classic Example: ISR → task
This is **the most common usage**.
- ISR detects interrupt
- Task processes data

```c
SemaphoreHandle_t xBinarySemaphore;

// Task:
void vTask(void *pvParameters)
{
    while (1)
    {
        // Wait for interrupt signal
        if (xSemaphoreTake(xBinarySemaphore, portMAX_DELAY) == pdTRUE)
        {
            // Process data
        }
    }
}

// ISR:
void EXTI_IRQHandler(void)
{
    BaseType_t xHigherPriorityTaskWoken = pdFALSE;

    xSemaphoreGiveFromISR(
        xBinarySemaphore,
        &xHigherPriorityTaskWoken
    );

    portYIELD_FROM_ISR(xHigherPriorityTaskWoken);
}
```

Execution Flow
```markdown
Create → Take (block) → ISR Give → Task wakes → Take succeeds → Run
```

#### Why `FromISR` version?
- Normal `xSemaphoreGive()` is NOT safe in ISR
- ISR must use special API


### Typical Mistakes
#### ❌ Forgetting initial give
If you don’t call:
```c
xSemaphoreGive(xBinarySemaphore);
```
the first `xSemaphoreTake()` will block forever.

#### ❌ Using normal API inside ISR
Always use:
- `xSemaphoreGiveFromISR()`
- `xQueueSendFromISR()`

## Counting Semaphore
A counting semaphore can hold a value 0 → max_count.

Instead of just 0 or 1 (like binary semaphore).

Used when:
- You need to count multiple events
- You manage multiple identical resources
- ISR produces events faster than task consumes

### `xSemaphoreCreateCounting()`
```c
SemaphoreHandle_t xSemaphoreCreateCounting(
    UBaseType_t uxMaxCount,
    UBaseType_t uxInitialCount
);
```

Example:
```c
SemaphoreHandle_t xCountingSemaphore;

void init(void)
{
    xCountingSemaphore = xSemaphoreCreateCounting(5, 0);  // Max value = 5, starts at 0
}
```

### `xSemaphoreGive()` 
- Increments count
- If tasks are waiting → unblocks one
- Cannot exceed `uxMaxCount`

### `xSemaphoreTake()`
- If count > 0 → decrement and continue
- If count == 0 → wait (depending on timeout)

### Classic Example 1: ISR Event Counting
- External interrupt occurs multiple times
- Task processes events slower than interrupt

```c
void EXTI_IRQHandler(void)   // Each interrupt → count++
{
    BaseType_t xHigherPriorityTaskWoken = pdFALSE;

    xSemaphoreGiveFromISR(
        xCountingSemaphore,
        &xHigherPriorityTaskWoken
    );

    portYIELD_FROM_ISR(xHigherPriorityTaskWoken);
}

void vTask(void *pvParameters)  // If interrupt happens 3 times, Task will run 3 times
{
    while (1)
    {
        xSemaphoreTake(xCountingSemaphore, portMAX_DELAY);

        // Process ONE event
    }
}
```

### Classic Example 2: Resource Pool Management
Imagine you have:
- 3 identical UART channels
- 3 identical buffers
- 3 identical hardware modules

```c
xSemaphoreCreateCounting(3, 3);  // 3 resources available initially

if (xSemaphoreTake(xSemaphore, portMAX_DELAY) == pdTRUE)  // Count becomes 2.
{
    // Use resource
}

xSemaphoreGive(xSemaphore);  // Count becomes 3 again.
```

## Mutex (Mutual Exclusion Semaphore)
A mutex is used to protect shared resources.

### `xSemaphoreCreateMutex(), xSemaphoreTake(), xSemaphoreGive()`
```c
SemaphoreHandle_t xMutex;

void init(void)
{
    xMutex = xSemaphoreCreateMutex();
}

void TaskA(void *pvParameters)
{
    while (1)
    {
        xSemaphoreTake(xMutex, portMAX_DELAY);

        printf("Task A running\n");

        xSemaphoreGive(xMutex);

        vTaskDelay(pdMS_TO_TICKS(1000));
    }
}
```
Unlike binary semaphore:
- Mutex starts in **available state**.
- The task that takes the mutex is also the task giving the mutex.

### Why `Mutex` Instead of Binary Semaphore


