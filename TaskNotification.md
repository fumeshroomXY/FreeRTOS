# Task Notification
Task Notification is a lightweight, fast, built-in mechanism for task-to-task or ISR-to-task communication in FreeRTOS.

Each task has a **32-bit notification value** inside its TCB (Task Control Block).

It can be used like:
- A binary semaphore
- A counting semaphore
- An event flag (bitwise)
- A small data mailbox (32-bit value)

It is **faster and more memory efficient** than queues or semaphores because no extra kernel object is needed.

## `xTaskNotify()`
Sends a 32-bit value to a task and optionally wakes it up.
```c
BaseType_t xTaskNotify(
    TaskHandle_t xTaskToNotify,
    uint32_t ulValue,              // the task's 32-bit notification value
    eNotifyAction eAction
);
```

### `ulValue` and `eNotifyAction`
| Mode                        | Behavior                        |
| --------------------------- | ------------------------------- |
| `eNoAction`                 | Just unblock task               |
| `eSetBits`                  | OR bits into notification value |
| `eIncrement`                | Increment value                 |
| `eSetValueWithOverwrite`    | Overwrite old value             |
| `eSetValueWithoutOverwrite` | Only set if no pending          |

Suppose:
```markdown
[ 00000000 00000000 00000000 00000000 ] is the task's notification value.
```
```c
xTaskNotify(taskHandle, ulValue, eAction);   ->  Modify that 32-bit box using this rule (eAction).
```

#### 
