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

Assume current value inside task is:
```markdown
0000 0000 0000 0000 0000 0000 0000 0010
```
```c
xTaskNotify(taskHandle, ulValue, eAction);   ->  Modify that 32-bit box using this rule (eAction).
```

#### eSetBits (Used like Event Flags)
```c
xTaskNotify(task, 0x01, eSetBits);
// Means: current_value |= ulValue
```
So the new value:
```markdown
0000 0000 0000 0000 0000 0000 0000 0011
```

#### eIncrement (Used like Counting Semaphore)
```c
xTaskNotify(task, 0, eIncrement);   // This ignores ulValue.
```
It simply does:
```c
current_value++
```
Used with `ulTaskNotifyTake()`

#### eSetValueWithOverwrite (Mailbox style)
```c
xTaskNotify(task, 100, eSetValueWithOverwrite);

// Means: current_value = ulValue
```
Used when sending data, like a 32-bit mailbox

#### eSetValueWithoutOverwrite
Same as above BUT:

If task hasn’t processed previous notification, it FAILS.

Used when you don't want to lose data.

#### eNoAction
Does NOT change value.

Only wakes task.


## When To Use What?
| Use Case           | API                                |
| ------------------ | ---------------------------------- |
| Wake task only     | xTaskNotifyGive                    |
| Counting semaphore | xTaskNotifyGive + ulTaskNotifyTake |
| Event bits         | xTaskNotify + xTaskNotifyWait      |
| Pass 32-bit data   | xTaskNotify + xTaskNotifyWait      |
| ISR wake-up        | vTaskNotifyGiveFromISR             |
