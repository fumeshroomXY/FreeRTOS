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
