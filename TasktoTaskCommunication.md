# Why Not Just Share Global Variables?
Example of BAD design:
```c
int sharedVar;

Task1: sharedVar++;
Task2: sharedVar--;
```

**Problem:**
- Context switch may happen in the middle
- Causes race condition
- Data corruption

# Task-to-task Communication Methods
In FreeRTOS, **Message Queue**, **Semaphore**, and **Event Group** are three core synchronization tools — but they solve different problems.

## Message Queue (Data Transfer Tool)
Transfer data safely between tasks (or ISR → task).

Like **a mailbox** that stores actual messages.

### Use When
- Task A produces data
- Task B processes data
- You need buffering

Example: Sensor task → Processing task
```c
QueueHandle_t xQueue;

xQueue = xQueueCreate(5, sizeof(int));

// Sender task
int value = 10;
xQueueSend(xQueue, &value, portMAX_DELAY);

// Receiver task
int received;
xQueueReceive(xQueue, &received, portMAX_DELAY);
```

### Characteristics
- Copies data into queue buffer
- FIFO behavior
- Can block when empty/full

## Semaphore (Signaling Tool)
Signal or control access — **no data transfer**

A door key — not a mailbox.

### A) Binary Semaphore
Used for **event notification**

Example:
- ISR signals task
- Task A wakes Task B

### B) Counting Semaphore
Used when:
- Multiple identical resources
- Counting events

### C) Mutex (Special Semaphore)
Used to protect shared resources.


## Event Group (Multi-Event Synchronization)
Wait for multiple conditions at once. Each bit represents one event.

A status board with many indicator LEDs.

Example:
```c
xEventGroupWaitBits(
    xEventGroup,
    BIT_0 | BIT_1,    // Wait until BIT_0 AND BIT_1 are both set
    pdTRUE,
    pdTRUE,
    portMAX_DELAY
);
```
