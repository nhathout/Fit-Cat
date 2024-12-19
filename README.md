# FitCat: Network of Smart Cat Collars with Activity Monitoring

**Date:** October 27, 2024

## Overview
This smart cat collar project is designed to capture, log, and transmit real-time activity data of cats. Using an accelerometer, a networked server, and an alphanumeric display, the collar determines whether a cat is sleeping, wandering, or performing a unique "moonwalk" activity. The collar displays its current activity on an alphanumeric display and communicates with a central server to participate in a leader determination system. Additionally, a button interface allows switching display modes, and a buzzer indicates leadership changes or continuous leadership status.

## Key Features

- **Activity Detection:**  
  Utilizes an ADXL343 accelerometer to detect and classify activities.  
  Supports states such as "Sleepy Time," "Wander Time," and "Moonwalk Time."

- **State Reporting:**  
  Shows the current activity state on a 14-segment alphanumeric display.  
  Sends real-time activity data and timestamps to a central server over Wi-Fi.

- **Leader Determination:**  
  Integrates with a central server to determine which cat is the "leader" based on accumulated active time over a rolling 10-minute window.  
  Responds to leader updates with a buzzer signal: continuous buzzing if currently the leader, and a single buzz on leader change events.

- **Button Interface:**  
  Cycles through multiple display modes at the press of a button:  
  - **Mode 0:** Default message ("Boots and Cats").  
  - **Mode 1:** Current activity state.  
  - **Mode 2:** Elapsed time in the current state.

- **Buzzing Mechanism:**  
  Continuous buzzing if the device’s cat is the leader.  
  A single buzz on receiving a leader change notification.

## Implementation Details

### Sensor Integration
- The ADXL343 accelerometer is used to read x, y, and z accelerations.
- Derived pitch and roll values help classify the cat’s current state.
- Thresholds and classification logic translate raw sensor readings into human-readable states.

### Display Interface
- A 14-segment alphanumeric display shows various messages:
  - Default greeting.
  - Current activity state.
  - Elapsed time in that state.
- I²C communication manages updates and scrolling text if needed.

### Network Communication
- Uses Wi-Fi to connect to a central server.
- Relies on both WebSockets and UDP for data exchange.
  - Activity data and timestamps are sent to the server.
  - Leader updates are received, prompting local buzzer events.

### Concurrency and Synchronization
- Multiple FreeRTOS tasks handle sensor data, display updates, button input, and network communication concurrently.
- Mutexes protect shared data to ensure thread-safe operations.
- Tasks are organized for predictable and stable behavior across all system functions.

## Solution Design

### Initialization (app_main)
- Initialize shared mutexes.
- Set up I2C and UART communication.
- Configure GPIOs for the button and buzzer.
- Connect to the Wi-Fi network.
- Initialize the accelerometer and prepare for sensor readings.
- Create and start FreeRTOS tasks for sensor reading, button handling, display updates, and network listening.

### Tasks and Functions

**Sensor Task (test_adxl343):**  
- Periodically reads accelerometer values (x, y, z).  
- Calculates roll, pitch, and overall acceleration to classify the cat’s activity state with `getCatState()`.  
- Updates shared state variables and resets timers as needed.

**Button Task (task_button_presses):**  
- Polls the button input.  
- On button press, cycles through display modes (0 → 1 → 2 → back to 0).  
- Uses a mutex to safely modify shared variables that determine the display content.

**Display Task (test_alpha_display):**  
- Updates the alphanumeric display based on the current display mode.  
- Mode 0: Shows "Boots and Cats".  
- Mode 1: Shows the cat’s current activity state.  
- Mode 2: Shows the elapsed time in the current activity state.  
- Handles scrolling for longer messages and communicates via I²C.

**Network Listener Task (network_listener_task):**  
- Listens for leader updates from the server using UDP.  
- On receiving a new leader ID, updates the local leader status and triggers the buzzer accordingly.

**WebSocket Event Handler (websocket_event_handler):**  
- On receiving data from the central server, parses the leader ID.  
- Updates buzzing state if this collar’s cat is the leader.  
- Implements a single buzz for leader change or continuous buzzing for ongoing leadership.

**Buzz Function (buzz):**  
- If `isBuzzing` is true and this cat is the leader, buzz continuously (on/off at intervals).  
- If a leader change occurred, buzz once.  
- Otherwise, ensure the buzzer remains off.

## Results and Learnings

**Achievements:**
- Successfully integrated sensor readings, network communications, and display output into one cohesive system.
- Achieved accurate classification of cat activities based on acceleration data.
- Established a leader determination mechanism that updates in real-time and is clearly signaled by the buzzer.

**Challenges:**
- Fine-tuning thresholds and logic for accurate activity classification proved time-consuming.  
- Managing concurrent tasks and ensuring thread-safe data access required careful synchronization.

**Potential Improvements:**
- Introduce hardware interrupts instead of polling for the button, improving efficiency.
- Refine circuit design to reduce complexity and improve reliability.
- Enhance power management for extended battery life in real-world scenarios.
- Include more robust error handling, especially around network communications.

## Demonstrations

- [**Device Functionality Demo Video**](https://drive.google.com/file/d/1tmxS8HGcO_dwnGMNcW7fP4neTti4bQX-/view?usp=sharing)
- [**Design Walkthrough Video**](https://drive.google.com/file/d/1KdEOFxNXSFyghgzOxwIRIRRMS36AR_G7/view?usp=sharing)

## Appendix

![circuit-diagram](https://github.com/nhathout/Fit-Cat/blob/main/CatCollarDiagram.jpg)

---

This smart cat collar prototype demonstrates a robust system capable of monitoring feline activity, displaying meaningful states on an alphanumeric display, and participating in a networked leader determination setup. Its extensible architecture, modular tasks, and well-defined communication patterns make it a strong foundation for future enhancements, fine-tuning, and real-world adoption.
