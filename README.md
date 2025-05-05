# Real-Time Room Monitoring System

A multithreaded environmental monitoring system for Raspberry Pi 4 using Rate Monotonic Scheduling (RMS). It reads data from gas, light, and motion sensors, applies real-time scheduling for deterministic behavior, and sends UDP telemetry for logging.

---

## Features

- Periodic data acquisition (gas, light, motion)
- POSIX `SCHED_FIFO` threads with RMS priority assignment
- Real-time alerts via GPIO (LED/Buzzer)
- UDP-based telemetry broadcasting
- POSIX timer activation (no `sleep()`)
- WCET, deadline miss, and jitter tracking
- Core affinity for thread isolation

---

## System Architecture

### Hardware Diagram

![image](https://github.com/user-attachments/assets/ee83d138-b29e-46c0-b955-c4690fbc23d7)

### Software Diagram

![image](https://github.com/user-attachments/assets/825bca96-b950-44d4-9ee1-1b3e3bd584cc)

---

## Hardware Components

| Component     | Function                                      | Interface                      |
|---------------|-----------------------------------------------|-------------------------------|
| MQ-135        | Gas sensor (CO₂, NH₃, pollutants)             | Analog via ADS1115 over I²C-1 |
| ADS1115       | 16-bit ADC for analog sensors                 | I²C-1 (GPIO2, GPIO3)           |
| BH1750        | Digital ambient light sensor                  | I²C-3 (GPIO4, GPIO5)           |
| PIR Sensor    | Motion detection                              | GPIO input (GPIO17)           |
| GPIO 23       | Alert signal output (LED/Buzzer)              | GPIO output                    |
| Ethernet/Wi-Fi| Data transmission over UDP                    | Socket interface               |

---

## Threads & Scheduling

| Thread                 | Function                                    | Period | WCET       | Priority |
|------------------------|---------------------------------------------|--------|------------|----------|
| `SensorSamplerThread`  | Reads MQ135, BH1750, PIR                    | 160 ms | ~10.675 ms | Highest  |
| `ControlServiceThread` | Checks thresholds, triggers GPIO alerts     | 200 ms | ~0.145 ms  | High     |
| `EnvironmentMonitorThread` | Classifies room state (safe, alert, etc.) | 220 ms | ~0.197 ms  | Medium   |
| `UDPSenderThread`      | Sends sensor data to a UDP receiver         | 240 ms | ~0.322 ms  | Lowest   |

Scheduling: `pthread_setschedparam()` with `SCHED_FIFO`  
Core Isolation: `pthread_setaffinity_np()` used to reduce jitter

---

## Real-Time Schedulability Analysis

- **Utilization:** ~8.24%
- **Liu & Layland Bound for 4 Tasks:** 75.683%
- **Cheddar Simulation Results:**
  - Context switches: 524
  - Preemptions: 0
  - Deadline misses: 0
  - Idle Time: 24,223 units

---

## Project Directory Structure

```bash
.
├── main.cpp
├── Makefile
├── README.md
├── images/
│   ├── Hardware_Diagram.png
│   └── SOFtware_BLOCK_DIAgram.png
├── common/
│   ├── GpioMmap.cpp
│   ├── GpioMmap.hpp
│   ├── SensorData.hpp
│   └── ThreadStats.hpp
├── sensors/
│   ├── ads1115.cpp / .hpp
│   ├── bh1750.cpp / .hpp
│   ├── mq135.cpp / .hpp
│   └── PIR.cpp / .hpp
├── threads/
│   ├── SensorSampler.cpp / .hpp
│   ├── ControlService.cpp / .hpp
│   ├── EnvironmentMonitor.cpp / .hpp
│   └── UDPSender.cpp / .hpp

### How to run

- run 'make'
- run 'sudo rtes_app'
