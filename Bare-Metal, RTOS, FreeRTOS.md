# Bare-Metal
A program that runs directly on hardware **without an operating system**.
```markdown
Hardware
   ↑
Your Application Code
```
There is:
- No task scheduler
- No threads
- No OS services
- No process isolation

You control Registers, Interrupts, Memory, Peripherals.

Example:
```c
int main(void)
{
    SystemInit();
    GPIO_Init();

    while (1)
    {
        LED_Toggle();
    }
}
```

## Use Bare-Metal When:
- Small MCU (low RAM/Flash)
- Simple control logic
- Very tight resource limits
- Bootloader
- Learning low-level concepts

# RTOS (Real-Time Operating System)
An **RTOS** is a lightweight operating system designed for real-time embedded systems.
```markdown
Hardware
   ↑
RTOS Kernel
   ↑
Multiple Tasks
```

The RTOS provides:
- Task scheduling
- Context switching
- Inter-task communication
- Timing services
- Synchronization (mutex, semaphore)

Example:
```c
void Task1(void *pvParameters)
{
    while (1)
    {
        LED_Toggle();
        vTaskDelay(1000);
        // Multiple tasks run
        // The RTOS scheduler decides which task runs
    }
}
```

## Why do we need an RTOS?
Bare-Metal works… until it gets complicated:
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
This works well when:
- Few tasks
- Simple timing
- Small project

But problems appear when the system grows.

### The Real Problem: Concurrency
Imagine your product needs:
- Sensor reading every 1 ms
- UART communication
- CAN bus handling
- Motor control
- Logging to Flash
- Bluetooth stack

In bare-metal, you must manually manage:
- State machines
- Timing counters
- Interrupt flags
- Priority handling

```c
if(flag1) { … }
if(flag2) { … }
if(timer_expired) { … }
```
- Hard to maintain
- Hard to scale
- Hard to debug
- Hard to prioritize

### What RTOS Solves
An RTOS (like FreeRTOS) provides:
#### 1. Task Scheduler
Instead of manually polling:
```markdown
Task A
Task B
Task C
```
You write:
```c
xTaskCreate(TaskA, ...);
xTaskCreate(TaskB, ...);
```
The RTOS automatically:
- Switches between tasks
- Manages priorities
- Handles preemption(a higher-priority task interrupts and takes CPU control from a lower-priority task)

#### 2. Deterministic Timing
You can write:
```c
vTaskDelayUntil(&lastWakeTime, 1);
```
Instead of manually checking timers.

#### 3. Priority Control
Example:
- Motor control = High priority
- Logging = Low priority

RTOS ensures:
- Critical tasks run first
- Low priority tasks don’t block important ones

Bare-metal requires you to design this manually.

#### 4. Clean Structure
Instead of one giant loop:
```c
while(1)
{
   everything mixed together
}
```
You get:
```markdown
Task_Sensor
Task_Comm
Task_Control
Task_UI
```
Much cleaner architecture.

### RTOS Has Cost
- More RAM (each task needs stack)
- More Flash
- Context switching overhead
- Added complexity

## Use RTOS When:
- Multiple independent tasks
- Communication stacks (CAN, TCP/IP)
- Complex state management
- Product-scale firmware

# RTOS vs Non-RTOS(general-purpose OS)
## What is an RTOS?
RTOS guarantees tasks are executed **within a predictable time limit**.

Examples: FreeRTOS, VxWorks, Zephyr

### Key Characteristic: Determinism

In RTOS:
- Task scheduling is predictable
- High-priority tasks can preempt lower ones
- Interrupt latency is controlled
- Worst-case execution time (WCET) matters

Used in:
- Motor control
- Automotive ECUs
- Industrial control
- Medical devices
- Aerospace

Because missing a deadline could mean:
- System failure
- Physical damage
- Safety hazard

## What is a Non-RTOS?
Non-RTOS focuses on throughput, fairness, and user experience — NOT strict timing guarantees.

Example: Linux, Windows, macOS

### Key Characteristic: No Hard Timing Guarantee
For example:

If you ask Linux to run something “every 1ms”,
it might run in 1ms, 2ms, 5ms or sometimes much later

That’s acceptable for:
- Browsers
- Office apps
- Video streaming
- Desktop software

But not acceptable for:
- Airbag deployment
- Anti-lock braking systems
- Motor PWM control


# Simple Rule of Thumb
| System Size                | Recommendation        |
| -------------------------- | --------------------- |
| Very small MCU             | Bare-metal            |
| Medium complexity          | RTOS                  |
| Large embedded Linux board | Full OS (e.g., Linux) |


# FreeRTOS
A small, lightweight Real-Time Operating System kernel for microcontrollers.

It runs on: ARM Cortex-M, RISC-V, AVR and many other MCUs.  
It is just a kernel, **not a full operating system with drivers and filesystem**.

## What Does FreeRTOS Provide?
### 1. Task Management
Each task:
- Has its own stack
- Has its own priority
- Runs “independently”

### 2. Scheduler
- Priority-based preemptive scheduling
- Higher priority task → runs first.

### 3. Inter-Task Communication
FreeRTOS provides:
- Queues
- Semaphores
- Mutexes
- Event groups
- Task notifications

Example (Queue):
```c
xQueueSend(queue, &data, 0);
xQueueReceive(queue, &data, portMAX_DELAY);
```

## Where FreeRTOS Is Common
- IoT devices
- Industrial controllers
- Automotive subsystems
- Communication stacks
- WiFi / Bluetooth modules

## FreeRTOS vs Linux (Very Important)
| Feature       | FreeRTOS          | Linux                |
| ------------- | ----------------- | -------------------- |
| Runs on MCU   | ✅ Yes             | ❌ No (needs MPU/MMU) |
| Deterministic | ✅ Yes             | ❌ Not strictly       |
| Memory size   | KB–MB             | Hundreds of MB       |
| Processes     | ❌ No (tasks only) | ✅ Yes                |



