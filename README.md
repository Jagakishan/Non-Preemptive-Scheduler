**Overview**
This project is RTOS style cooperative(non-preemptive) scheduler that does sensor acquisition, scaling raw sensor values, transmission of data over CAN bus and logging scaled data for reference in periodic fashion.
The system reads data from multislave sensors using a single I2C channel, computes scaled readable values in a seperate abstraction layer as it involves floating point math and cpu heavy math. Then transmits the 
data over CAN bus which uses twisted pair cable to another node where the data is stored in a SPI-based flash in a circular fashion(oldest sector gets overwritten first). Apart from this, for debugging purpose
a non-blocking UART is designed. The reason it being non-blocking is because it shouldn't hold CPU while high priority processes are waiting. I will give a elaborate explanation down.

**Why No RTOS/FreeRTOS?**
I haven't used FreeRTOS or any pre-built RTOS in this project. One reason being I have absolutely no idea how to use it and another reason is that using FreeRTOS for a cooperative scheduler is too much. So I just 
scheduled everything on my own and made sure no process has to wait long to get executed.

**CAN Integration**
I have to tell this early. The CAN functionality is implemented as a standalone experiment in "Playing arount STM" repo due to the lack of a CAN transceiver in the current hardware setup. I have one(mcp2551 miniaturized ic like transceiver), but that's not working. So, I have put together a minimal initialization, data structuring, filter configuration code there. I does not have any relevance to this project.
That's why the reading and logging code is put in the same "main.c" file.

**Key Features**

**1.Cooperative (non-preemptive) task scheduling**
- Time-based task execution using SysTick
- Deterministic execution without blocking delays

**2.Non-blocking UART debugging**
- Custom _write() implementation for printf
- Interrupt-driven transmission (no busy waits)(not the usual one you see around online)
- One limitation is that it raises an interrupt after printing every character, but that's the core idea of this being non-blocking. It checks if there's any other process waiting after printing every character. If there's any, it hands over CPU.
- Safe to use inside periodic tasks

**3.Multi-slave I2C sensor interface**
- MPU6050 (IMU) support
- BMP280 (Pressure & Temperature) support
- Proper initialization, timing separation, and bus recovery
- Address based slave communication
- Sensor data abstraction
- Raw data stored in structured formats
- Optional scaling/compensation performed outside time-critical paths(only while transitting or debugging to reduce CPU load)

**4.SPI-ready architecture**
- Designed for logging sensor data into external SPI flash (W25Q series-Winbond)(I have used the 128Mb one, but that doesn't matter as long as you don't want your sensor values for so long time)
- Logging task fits naturally into the scheduler
- Logged in a circular fashion in First in First overwritten fashion.
- I have a seperate repository for this Winbond flashes that has the code(easy to use functions) for talking to its registers. If you're interested, go ahead and take a look. But it won't be flashy, just have enough functions to cover all functionality.

**5.CAN-based distributed system (separate repo)**
- Sensor data packetization
- CAN transmission to another node
- Data reception and storage in flash
- Kept separate due to transceiver unavailability(The mcp2551 part I bought is not working. If you got one, do help me)


## Author
Jagakishan S.K
