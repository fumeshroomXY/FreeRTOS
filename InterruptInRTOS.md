# Interrupts in Bare-Metal
Bare-metal means:
- No OS
- Your `main()` runs forever
- You manage everything manually
```c
int main(void)
{
    init();

    while (1)
    {
        task1();
        task2();
    }
}
```

## How Interrupt Works in Bare-Metal
When an interrupt occurs:
- CPU stops current code
- Context (PC, registers) is pushed to stack
- **ISR (Interrupt Service Routine) runs**
- CPU restores context
- Execution continues where it stopped

### Important Characteristics
- No task switching
- ISR must be short
- You manually manage shared variables
- You must disable/enable interrupts carefully

# Interrupts in RTOS
In RTOS:
- Interrupts can trigger **task switching**
- ISR can wake up a higher priority task
- Scheduler may run immediately after ISR

## How It Works in FreeRTOS
When interrupt happens:
1. CPU jumps to ISR (same as bare-metal)
2. ISR may:
    - Send data to queue
    - Give semaphore
    - Notify a task
3. If **a higher priority task** waiting for something becomes ready state because that event happens
4. Context switch happens immediately (after ISR exits)

So interrupt can cause **preemption indirectly**.

## ISR
Instead of doing heavy work in ISR, we:
- Signal a task
- Let scheduler run the correct task

This keeps:
- **Interrupt latency low**
- System deterministic
- **Design clean**

## Key Difference
| Feature                  | Bare-Metal | RTOS              |
| ------------------------ | ---------- | ----------------- |
| Task switching           | ❌ No       | ✅ Yes             |
| Scheduler                | ❌ None     | ✅ Yes             |
| ISR can wake task        | ❌ No       | ✅ Yes             |
| Context switch after ISR | ❌ No       | ✅ Yes (if needed) |


## Example Comparison
### Bare-Metal Version
Interrupt sets a flag:
```c
void UART_IRQHandler(void)
{
    rx_flag = 1;
}

int main(){
  while (1)
  {
      if (rx_flag)
      {
          process_data();
          rx_flag = 0;
      }
  }
}
```
CPU wastes time polling.


### FreeRTOS Version
ISR gives semaphore:
```c
void UART_IRQHandler(void)
{
    BaseType_t xHigherPriorityTaskWoken = pdFALSE;

    xSemaphoreGiveFromISR(xSemaphore, &xHigherPriorityTaskWoken);

    portYIELD_FROM_ISR(xHigherPriorityTaskWoken);
}

// Task blocks:
void UartTask(void *pvParameters)
{
    while (1)
    {
        xSemaphoreTake(xSemaphore, portMAX_DELAY);
        process_data();
    }
}
```
CPU sleeps until interrupt occurs, **efficient and deterministic**

### Timing Behavior
**Bare-Metal:**  
Interrupt preempts main loop only.
```markdown
Low task running
→ Interrupt
→ ISR
→ Return to low task
```

**RTOS:**  
Interrupt may:
- Preempt a low priority task
- Wake high priority task
- Trigger immediate context switch

So actual order could be:
```markdown
Low task running
→ Interrupt
→ ISR gives semaphore
→ High task becomes Ready
→ ISR exits
→ Context switch
→ High task runs immediately
```

