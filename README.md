# 🧠 STM32 FreeRTOS Queue Demo on STM32F4 using Renode

This project demonstrates **inter-task communication using FreeRTOS queues** on an **STM32F4** microcontroller, simulated in **Renode**.  
It shows how a **Sender Task** transmits integer data to a **Receiver Task** via a queue, and how queue parameters can be observed in memory during simulation.

---

## 🚀 Project Overview

| Component | Description |
|------------|-------------|
| **MCU** | STM32F407VG (Cortex-M4) |
| **RTOS** | FreeRTOS |
| **Toolchain** | STM32CubeIDE |
| **Simulator** | Renode |
| **Communication** | UART2 (printf redirected) |

---

## ⚙️ Features

- Implements **FreeRTOS queue communication** between two tasks:
  - **Sender Task:** periodically sends integer values (1, 2, 3, …) to the queue.
  - **Receiver Task:** receives and prints values from the queue via UART.
- Demonstrates:
  - Task scheduling
  - Queue creation and usage (`xQueueCreate`, `xQueueSend`, `xQueueReceive`)
  - Real-time queue monitoring in Renode
- Supports UART output redirection via `printf()`

---

## 🧩 Project Structure

Queue_API_FreeRTOS_Demo/
│
├── Core/
│ ├── Inc/
│ │ └── main.h
│ └── Src/
│ └── main.c ← main application logic
│
├── Drivers/
│ └── STM32F4xx_HAL_Driver/
│
├── Middlewares/
│ └── Third_Party/FreeRTOS/
│
├── queue_demo.repl ← Renode simulation script
└── README.md



## 📜 Main Code Summary

### Queue Initialization

dataQueue = xQueueCreate(10, sizeof(int));
if (dataQueue == NULL) {
    printf("Queue creation failed!\r\n");
    while(1);
}
Sender Task
c
Copy code
void SenderTask(void *pvParameters)
{
    int value = 0;
    while(1)
    {
        value++;
        xQueueSend(dataQueue, &value, portMAX_DELAY);
        printf("[Sender] Sent: %u, Messages in Queue: %lu\r\n",
               value, uxQueueMessagesWaiting(dataQueue));
        vTaskDelay(pdMS_TO_TICKS(1000));
    }
}
Receiver Task
c
Copy code
void ReceiverTask(void *pvParameters)
{
    int receivedValue;
    while(1)
    {
        if(xQueueReceive(dataQueue, &receivedValue, portMAX_DELAY))
        {
            printf("[Receiver] Received: %u, Messages left: %lu\r\n",
                   receivedValue, uxQueueMessagesWaiting(dataQueue));
        }
    }
}
🧠 Understanding FreeRTOS Queue Functions
Function	Purpose
xQueueCreate(length, itemSize)	Create a queue that can store length items of size itemSize
xQueueSend(queue, *item, timeout)	Send an item to the queue (wait if full)
xQueueReceive(queue, *item, timeout)	Receive an item from the queue (wait if empty)
uxQueueMessagesWaiting(queue)	Returns number of items currently stored
xQueueSendFromISR()	Queue write from ISR context
vQueueDelete()	Delete a queue when no longer needed

🧪 Running in Renode
1️⃣ Load the ELF
(monitor) mach create
(machine-0) machine LoadPlatformDescription @platforms/boards/stm32f4_discovery.repl
(machine-0) sysbus LoadELF @Queue_API_FreeRTOS_Demo.elf
2️⃣ Start the Simulation
(machine-0) start
3️⃣ View UART Output
(machine-0) showAnalyzer uart2
You should see:

<img width="1908" height="1020" alt="Screenshot 2025-10-18 200224" src="https://github.com/user-attachments/assets/f348d85f-99e2-43c6-9095-3e50a2f40b3b" />

[Sender] Sent: 1, Messages in Queue: 1
[Receiver] Received: 1, Messages left: 0
[Sender] Sent: 2, Messages in Queue: 1
[Receiver] Received: 2, Messages left: 0
🧭 Memory Monitoring in Renode
Find the Queue Variable Address
Use the ELF map:

.bss.dataQueue  0x200000d0  0x4  ./Core/Src/main.o
Watch the Queue Pointer
bash
Copy code
(monitor) sysbus LogMemoryAccess 0x200000D0 0x4
Dump Queue Memory
bash
Copy code
(monitor) mem Dump 0x200000D0 32
📦 Build & Flash (optional)
If running on actual STM32F4 Discovery hardware:

Build the project in STM32CubeIDE.

Connect the board via ST-Link.

Click Run → Debug As → STM32 MCU C/C++ Application.

📘 References
FreeRTOS Queue API Documentation

Renode Official Docs

STM32F4 Reference Manual

🧑‍💻 Author
Arjun (github.com/arjun123445678)
Developed as a learning project for FreeRTOS Inter-Task Communication using STM32 + Renode.

🪶 License
This project is licensed under the MIT License — see the LICENSE file for details.


Would you like me to also create a **`queue_demo.repl`** Renode platform script (ready-to-run with UART2 analyzer + ELF load + start commands)?  
It’ll make your GitHub repo fully runnable in Renode with one click.
