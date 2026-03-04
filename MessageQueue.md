# Message Queue
Transfer data safely between tasks (or ISR → task).

Like **a mailbox** that stores actual messages.

- Works like a FIFO buffer
- Has a fixed length
- Copies data into queue buffer
- Can block when empty/full

When queue is empty/full:
- Task moves to **Blocked state**
- Added to queue’s waiting list
- Automatically woken when condition changes

## `xQueueCreate()`
Create a queue object in RAM.
```c
QueueHandle_t xQueueCreate(
    UBaseType_t uxQueueLength,   // Maximum number of items the queue can hold
    UBaseType_t uxItemSize       // Size (in bytes) of each item
);
```

Example:
```c
QueueHandle_t xQueue;

xQueue = xQueueCreate(
            5,              // queue can hold 5 items
            sizeof(int)     // each item is an int
         );
```

## `xQueueSend()`
Send (copy) data to the queue.
```c
BaseType_t xQueueSend(
    QueueHandle_t xQueue,
    const void * pvItemToQueue,   // Pointer to data to copy into queue
    TickType_t xTicksToWait       // How long to wait if queue is full
);
```
Example:
```c
int value = 10;

xQueueSend(
    xQueue,
    &value,
    portMAX_DELAY
);
```

## `xQueueReceive()`
Receive (copy out) data from queue.
```c
BaseType_t xQueueReceive(
    QueueHandle_t xQueue,
    void * pvBuffer,           // Where received data will be stored
    TickType_t xTicksToWait    // How long to wait if queue is full
);
```

Example:
```c
int received;

xQueueReceive(
    xQueue,
    &received,
    portMAX_DELAY
);
```

## Complete Working Example
```c
QueueHandle_t xQueue;

void vProducer(void *pvParameters)
{
    int counter = 0;

    while(1)
    {
        xQueueSend(xQueue, &counter, portMAX_DELAY);
        counter++;
        vTaskDelay(pdMS_TO_TICKS(1000));
    }
}

void vConsumer(void *pvParameters)
{
    int data;

    while(1)
    {
        xQueueReceive(xQueue, &data, portMAX_DELAY);
        printf("Received: %d\n", data);
    }
}

int main(void)
{
    xQueue = xQueueCreate(5, sizeof(int));

    xTaskCreate(vProducer, "Producer", 1000, NULL, 1, NULL);
    xTaskCreate(vConsumer, "Consumer", 1000, NULL, 1, NULL);

    vTaskStartScheduler();

    while(1);
}
```


## Important Notes
1. ISR must use `xQueueSendFromISR()` and `xQueueReceiveFromISR()`
2. Send Pointer Instead when passing large structs can be inefficient


Example:
```c
typedef struct {
    uint8_t image[1024];    // 5 × 1024 = 5120 bytes
} BigData_t;

BigData_t bigData;

xQueue = xQueueCreate(
            5,
            sizeof(BigData_t *)
         );

BigData_t *pData = &bigData;
xQueueSend(xQueue, &pData, portMAX_DELAY);

BigData_t *pReceived;
xQueueReceive(xQueue, &pReceived, portMAX_DELAY);
```

