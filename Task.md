# Function in Bare-Metal
In a **bare-metal** system (no OS), your program typically looks like this:
```c
int main(void)
{
    init();

    while (1)
    {
        task1();
        task2();
        task3();
    }
}
```

**What is a “function” here?**:
- Runs when called
- Executes to completion(no infinite loop in the function)
- No automatic scheduling
- No independent stack
- No context switching(Shares the same CPU context as everything else)
- No priority
- If `task1()` blocks → everything blocks.

# Task in FreeRTOS
In FreeRTOS, instead of manually calling functions in a loop, you create **tasks**.
Example:
```c
void Task1(void *pvParameters)
{
    while (1)  // Infinite loop is expected.
    {
        do_something();
        vTaskDelay(pdMS_TO_TICKS(100));
    }
}
```

Then:
```c
xTaskCreate(Task1, "T1", 1000, NULL, 1, NULL);
vTaskStartScheduler();
```

**What is a Task?**:
- A function + its own stack + never returns
- Independently scheduled
- Managed by the kernel
- Can be preempted
- Has priority

## Why Tasks Must Have while(1)
Because when a task function returns:
- Its stack is destroyed
- FreeRTOS deletes it

## Core Differences
| Aspect            | Bare-metal Function    | FreeRTOS Task               |
| ----------------- | ---------------------- | --------------------------- |
| Execution         | Called manually        | Scheduled by kernel         |
| Stack             | Shared main stack      | Each task has its own stack |
| Preemption        | No                     | Yes (if enabled)            |
| Priority          | No                     | Yes                         |
| Blocking          | Must not block         | Can block safely            |
| Context switching | No                     | Yes                         |
| Concurrency       | Manual (state machine) | Kernel managed              |

## `xTaskCreate()`
xTaskCreate() is the FreeRTOS API used to:
- Create a new task
- Allocate its stack
- Initialize its control block (TCB)
- Register it to the scheduler
- **NOT** start running the task immediately.

The task only starts running after:
```c
vTaskStartScheduler();
```

At that moment:
- First context switch occurs
- Main stack is replaced by first task stack
- Scheduler takes control

### Parameters
| # | Parameter       | Type                     | What It Represents           | Important Notes                                 | Example        |
| - | --------------- | ------------------------ | ---------------------------- | ----------------------------------------------- | -------------- |
| 1 | `pxTaskCode`    | `TaskFunction_t`         | Pointer to the task function | Must be of type `void task(void *pvParameters)` | `MyTask`       |
| 2 | `pcName`        | `const char *`           | Task name (for debugging)    | Used in debugger / trace tools only             | `"SensorTask"` |
| 3 | `uxStackDepth`  | `configSTACK_DEPTH_TYPE` | Stack size for this task     | Measured in **words**, not bytes                | `128`         |
| 4 | `pvParameters`  | `void *`                 | Pointer passed to task       | Can pass struct, int pointer, etc.              | `&myData`      |
| 5 | `uxPriority`    | `UBaseType_t`            | Task priority                | Higher value = higher priority                  | `2`            |
| 6 | `pxCreatedTask` | `TaskHandle_t *`         | Optional handle to the task  | Needed if you want to control task later        | `&taskHandle`  |

#### `pvParameters`
Example:
```c
typedef struct
{
    int id;
    int value;
} TaskData;

TaskData data = {1, 100};

xTaskCreate(MyTask, "T1", 1000, &data, 1, NULL);

// Inside Task:
void MyTask(void *pvParameters)
{
    ...
    TaskData *p = (TaskData*) pvParameters;
    ...
}
```

#### `pxCreatedTask`
If you want to suspend/delete/notify:
```c
TaskHandle_t myHandle;
xTaskCreate(..., &myHandle);
```



### Example Full Flow
```c
int main(void)
{
    init();

    xTaskCreate(Task1, "T1", 1000, NULL, 1, NULL);
    xTaskCreate(Task2, "T2", 1000, NULL, 2, NULL);

    vTaskStartScheduler();

    while(1); // should never reach here
}
```
After scheduler starts:
- Highest priority ready task runs first
- Tasks switch via:
    - tick interrupt
    - blocking calls
    - priority preemption
