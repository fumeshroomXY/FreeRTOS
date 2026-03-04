# Event Group
In FreeRTOS, an Event Group is a **synchronization mechanism** that uses **bits (flags)** to signal events between tasks or between ISR and tasks.

Used when:
- A task needs to wait for **multiple conditions**
- Multiple tasks need to be notified about an event
- You want to **replace many semaphores** with a cleaner solution
- You need **bitwise event synchronization**

## Each bit represents an event
```c
EventBits_t eventGroup;
```
| Bit  | Meaning Example |
| ---- | --------------- |
| BIT0 (1 << 0) | WiFi connected  |
| BIT1 (1 << 1)| Data received   |
| BIT2 (1 << 2)| Sensor ready    |
| BIT3 (1 << 3)| Error occurred  |

Multiple bits can be set at once.


## `xEventGroupCreate()`
Creates an event group object.
```c
EventGroupHandle_t xEventGroupCreate(void);  // Initializes all bits to 0 and returns a handle
```

## `xEventGroupSetBits()`
Sets one or more bits in the event group.
```c
EventBits_t xEventGroupSetBits(
    EventGroupHandle_t xEventGroup,
    const EventBits_t uxBitsToSet
);
```
Example:
```c
#define WIFI_READY   (1 << 0)
#define SENSOR_READY (1 << 1)

xEventGroupSetBits(xEventGroup, WIFI_READY | SENSOR_READY);
```

## `xEventGroupWaitBits()`
Wait (block) until specific bits are set.
```c
EventBits_t xEventGroupWaitBits(
    EventGroupHandle_t xEventGroup,
    const EventBits_t uxBitsToWaitFor,
    const BaseType_t xClearOnExit,
    const BaseType_t xWaitForAllBits,
    TickType_t xTicksToWait
);
```
### `uxBitsToWaitFor`
Which bits you care about.
```c
WIFI_READY | SENSOR_READY
```

### `xClearOnExit`
- `pdTRUE` → Clear bits automatically after waking
- `pdFALSE` → Leave bits set

### `xWaitForAllBits`
- `pdTRUE` → Wait for ALL bits until they are set
- `pdFALSE` → Wait for ANY bit

## Complete Example
```c
#define WIFI_READY_BIT    (1 << 0)
#define SENSOR_READY_BIT  (1 << 1)

EventGroupHandle_t xSystemEvents;

int main(void)
{
    xSystemEvents = xEventGroupCreate();

    xTaskCreate(vWiFiTask, "WiFi", 1000, NULL, 2, NULL);
    xTaskCreate(vSensorTask, "Sensor", 1000, NULL, 2, NULL);
    xTaskCreate(vAppTask, "App", 1000, NULL, 1, NULL);

    vTaskStartScheduler();
}

void vWiFiTask(void *pvParameters)
{
    printf("WiFi: Initializing...\n");

    vTaskDelay(pdMS_TO_TICKS(2000));  // simulate delay

    printf("WiFi: Connected!\n");

    // Set WIFI_READY_BIT
    xEventGroupSetBits(xSystemEvents, WIFI_READY_BIT);

    vTaskDelete(NULL);
}

void vSensorTask(void *pvParameters)
{
    printf("Sensor: Initializing...\n");

    vTaskDelay(pdMS_TO_TICKS(1000));  // simulate delay

    printf("Sensor: Ready!\n");

    // Set SENSOR_READY_BIT
    xEventGroupSetBits(xSystemEvents, SENSOR_READY_BIT);

    vTaskDelete(NULL);
}

void vAppTask(void *pvParameters)
{
    printf("App: Waiting for system ready...\n");

    xEventGroupWaitBits(
        xSystemEvents,
        WIFI_READY_BIT | SENSOR_READY_BIT,  // wait for both
        pdTRUE,     // clear bits when exit
        pdTRUE,     // wait for ALL bits
        portMAX_DELAY
    );

    printf("App: System Ready! Start sending data...\n");

    while (1)
    {
        printf("App: Sending data...\n");
        vTaskDelay(pdMS_TO_TICKS(3000));
    }
}
```


## `xEventGroupSync()`
It is used when multiple tasks must **reach the same point** before continuing.

= Set my bit + Wait for all required bits
```
EventBits_t xEventGroupSync(
    EventGroupHandle_t xEventGroup,
    const EventBits_t uxBitsToSet,
    const EventBits_t uxBitsToWaitFor,
    TickType_t xTicksToWait
);
```

Example:
```c
#define TASK1_BIT (1 << 0)
#define TASK2_BIT (1 << 1)
#define TASK3_BIT (1 << 2)

// Task 1
xEventGroupSync(
    xEventGroup,
    TASK1_BIT,
    TASK1_BIT | TASK2_BIT | TASK3_BIT,
    portMAX_DELAY
);

// Task 2
xEventGroupSync(
    xEventGroup,
    TASK2_BIT,
    TASK1_BIT | TASK2_BIT | TASK3_BIT,
    portMAX_DELAY
);

// Task 3
xEventGroupSync(
    xEventGroup,
    TASK3_BIT,
    TASK1_BIT | TASK2_BIT | TASK3_BIT,
    portMAX_DELAY
);
```

### What Happens Internally
1. Task sets its own bit

2. Then waits until all bits are set

3. When last task arrives → all tasks **unblock together**

This is called a **Barrier synchronization**
