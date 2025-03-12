# HLK-LD2450 1T2R 24G Multi-Target Status Detection Radar Communication Protocol

## Chapter 2: Communication Protocol

This chapter describes the communication protocol used by the HLK-LD2450 radar module. It covers the protocol format, command structure with ACK responses, radar data output, and the command configuration method. The radar communicates via a TTL-level serial port with the following default settings: baud rate 256000, 1 stop bit, and no parity.

---

## 2.1 Protocol Format

### 2.1.1 Protocol Data Format

The HLK-LD2450 module uses little-endian format for serial data communication. All data in the tables below are represented in hexadecimal.

### 2.1.2 Command Protocol Frame Format

The command configuration and ACK frames are defined as follows:

#### Table 2: Transmitted Command Protocol Frame Format

| Field         | Description                    |
|---------------|--------------------------------|
| **Frame Header**  | FD FC FB FA                  |
| **Data Length**   | 2 bytes (refer to Table 3)   |
| **Frame Data**    | See Table 3                  |
| **Frame Tail**    | 04 03 02 01                 |

#### Table 3: Transmitted Frame Data Format

| Field         | Description           |
|---------------|-----------------------|
| **Command Word**  | 2 bytes               |
| **Command Value** | N bytes               |

#### Table 4: ACK Command Protocol Frame Format

| Field         | Description                    |
|---------------|--------------------------------|
| **Frame Header**  | FD FC FB FA                  |
| **Data Length**   | 2 bytes (refer to Table 5)   |
| **Frame Data**    | See Table 5                  |
| **Frame Tail**    | 04 03 02 01                 |

#### Table 5: ACK Frame Data Format

| Field                    | Description                                            |
|--------------------------|--------------------------------------------------------|
| **Transmitted Command**  | 0x0100 (2 bytes) followed by the return value (N bytes) |

---

## 2.2 Sending Commands and ACK

### 2.2.1 Enable Configuration Command

*Before any other command is executed, this command must be sent; otherwise, other commands will be ineffective.*

- **Command Word:** 0x00FF  
- **Command Value:** 0x0001  
- **Return Value:**  
  - 2-byte ACK status (0: success, 1: failure)  
  - 2-byte protocol version (0x0001)  
  - 2-byte buffer size (0x0040)

**Send Data:**

```
FD FC FB FA 04 00 FF 00 01 00 04 03 02 01
```

**Radar ACK (Success):**

```
FD FC FB FA 08 00 FF 01 00 00 01 00 40 00 04 03 02 01
```

---

### 2.2.2 End Configuration Command

*This command ends the configuration process and returns the radar to working mode. To send further commands, you must re-enable configuration first.*

- **Command Word:** 0x00FE  
- **Command Value:** None  
- **Return Value:** 2-byte ACK status (0: success, 1: failure)

**Send Data:**

```
FD FC FB FA 02 00 FE 00 04 03 02 01
```

**Radar ACK (Success):**

```
FD FC FB FA 04 00 FE 01 00 00 04 03 02 01
```

---

### 2.2.3 Single Target Tracking

*Sets the radar to single target tracking mode.*

- **Command Word:** 0x0080  
- **Command Value:** None  
- **Return Value:** 2-byte ACK status (0: success, 1: failure)

**Send Data:**

```
FD FC FB FA 02 00 80 00 04 03 02 01
```

**Radar ACK (Success):**

```
FD FC FB FA 04 00 80 01 00 00 04 03 02 01
```

---

### 2.2.4 Multi-Target Tracking

*Sets the radar to multi-target tracking mode.*

- **Command Word:** 0x0090  
- **Command Value:** None  
- **Return Value:** 2-byte ACK status (0: success, 1: failure)

**Send Data:**

```
FD FC FB FA 02 00 90 00 04 03 02 01
```

**Radar ACK (Success):**

```
FD FC FB FA 04 00 90 01 00 00 04 03 02 01
```

---

### 2.2.5 Query Target Tracking Mode

*Queries the current target tracking mode of the module. The default mode is multi-target tracking.*

- **Command Word:** 0x0091  
- **Command Value:** None  
- **Return Value:**  
  - 2-byte ACK status (0: success, 1: failure)  
  - 2-byte tracking mode value:  
    - 0x0001 for single target tracking  
    - 0x0002 for multi-target tracking

**Send Data:**

```
FD FC FB FA 02 00 91 00 04 03 02 01
```

**Radar ACK (Success):**

For single target tracking:

```
FD FC FB FA 06 00 91 01 00 00 01 00 04 03 02 01
```

For multi-target tracking:

```
FD FC FB FA 06 00 91 01 00 00 02 00 04 03 02 01
```

---

### 2.2.6 Read Firmware Version Command

*Reads the radar firmware version information.*

- **Command Word:** 0x00A0  
- **Command Value:** None  
- **Return Value:**  
  - 2-byte ACK status (0: success, 1: failure)  
  - 2-byte firmware type (0x0000)  
  - 2-byte main version number  
  - 4-byte sub-version number

**Send Data:**

```
FD FC FB FA 02 00 A0 00 04 03 02 01
```

**Radar ACK (Success):**

```
FD FC FB FA 0C 00 A0 01 00 00 00 00 02 01 16 24 06 22 04 03 02 01
```

*The corresponding firmware version is **V1.02.22062416**.*

---

### 2.2.7 Set Serial Port Baud Rate

*Sets the serial port baud rate. The configuration is saved and takes effect after a module restart.*

- **Command Word:** 0x00A1  
- **Command Value:** 2-byte baud rate selection index  
- **Return Value:** 2-byte ACK status (0: success, 1: failure)

#### Table 6: Serial Port Baud Rate Selection

| Baud Rate Selection Index | Baud Rate |
|---------------------------|-----------|
| 0x0001                    | 9600      |
| 0x0002                    | 19200     |
| 0x0003                    | 38400     |
| 0x0004                    | 57600     |
| 0x0005                    | 115200    |
| 0x0006                    | 230400    |
| 0x0007                    | 256000    |
| 0x0008                    | 460800    |

*The factory default is **0x0007 (256000)**.*

**Send Data:**

```
FD FC FB FA 04 00 A1 00 07 00 04 03 02 01
```

**Radar ACK (Success):**

```
FD FC FB FA 04 00 A1 01 00 00 04 03 02 01
```

---

### 2.2.8 Restore Factory Settings

*Restores all configuration values to the factory defaults. The changes take effect after a module restart.*

- **Command Word:** 0x00A2  
- **Command Value:** None  
- **Return Value:** 2-byte ACK status (0: success, 1: failure)

**Send Data:**

```
FD FC FB FA 02 00 A2 00 04 03 02 01
```

**Radar ACK (Success):**

```
FD FC FB FA 04 00 A2 01 00 00 04 03 02 01
```

#### Table 7: Factory Default Configuration Values

| Configuration Item       | Default Value |
|--------------------------|---------------|
| Serial Port Baud Rate    | 256000        |
| Bluetooth Switch         | On            |
| Tracking Mode            | Multi-target  |
| Region Filtering Feature | Off           |

---

### 2.2.9 Restart Module

*Upon receiving this command, the module will automatically restart after sending the ACK.*

- **Command Word:** 0x00A3  
- **Command Value:** None  
- **Return Value:** 2-byte ACK status (0: success, 1: failure)

**Send Data:**

```
FD FC FB FA 02 00 A3 00 04 03 02 01
```

**Radar ACK (Success):**

```
FD FC FB FA 04 00 A3 01 00 00 04 03 02 01
```

---

### 2.2.10 Bluetooth Settings

*Controls the enabling or disabling of the Bluetooth feature. Bluetooth is enabled by default. The setting is persistent after power-off and takes effect after a restart.*

- **Command Word:** 0x00A4  
- **Command Value:**  
  - 0x0100 to enable Bluetooth  
  - 0x0000 to disable Bluetooth  
- **Return Value:** 2-byte ACK status (0: success, 1: failure)

**Send Data (to enable Bluetooth):**

```
FD FC FB FA 04 00 A4 00 01 00 04 03 02 01
```

**Radar ACK (Success):**

```
FD FC FB FA 04 00 A4 01 00 00 04 03 02 01
```

---

### 2.2.11 Get MAC Address

*Queries the module’s MAC address.*

- **Command Word:** 0x00A5  
- **Command Value:** 0x0001  
- **Return Value:**  
  - 2-byte ACK status (0: success, 1: failure)  
  - 1-byte fixed type (0x00)  
  - 3-byte MAC address (big-endian)

**Send Data:**

```
FD FC FB FA 04 00 A5 00 01 00 04 03 02 01
```

**Radar ACK (Success):**

```
FD FC FB FA 0A 00 A5 01 00 00 8F 27 2E B8 0F 65 04 03 02 01
```

*The retrieved MAC address is **8F 27 2E B8 0F 65**.*

---

### 2.2.12 Query Current Region Filtering Configuration

*Queries the current region filtering configuration of the module.*

- **Command Word:** 0x00C1  
- **Command Value:** None  
- **Return Value:**  
  - 2-byte ACK status (0: success, 1: failure)  
  - 2-byte region filtering type  
  - 24-byte region coordinate configuration

**Region Filtering Details:**

- **Region Filtering Type:**  
  - 0: Disable region filtering  
  - 1: Only detect targets within the specified region  
  - 2: Do not detect targets within the specified region

- **Region Coordinates:**  
  For each region (up to three), a rectangular area is defined by the coordinates of two diagonal vertices. Each vertex has an x and y coordinate in signed int16 format (unit: mm). A coordinate value of 0 indicates the region is not used.

**Send Data:**

```
FD FC FB FA 02 00 C1 00 04 03 02 01
```

**Radar ACK (Success):**

```
FD FC FB FA 1E 00 C1 01 00 00 01 00 E803 E803 18FC 8813 0000 0000 0000 0000 0000 0000 0000 0000 04 03 02 01
```

*This indicates that the current configuration is set to **only detect targets within** the rectangular region defined by the diagonal vertices **(1000, 1000)** and **(-1000, 5000)**.*

---

### 2.2.13 Set Region Filtering Configuration

*Sets the module’s region filtering configuration. The configuration is saved after power-off and takes effect immediately.*

- **Command Word:** 0x00C2  
- **Command Value:** 26-byte region filtering configuration value (format as per Table 8: Region Filtering Configuration Value Format)  
- **Return Value:** 2-byte ACK status (0: success, 1: failure)

**Send Data:**

```
FD FC FB FA 1C 00 C2 00 02 00 E803 E803 18FC 8813 0000 0000 0000 0000 0000 0000 0000 0000 04 03 02 01
```

*This indicates the configuration is set to **not detect targets within** the rectangular region defined by the diagonal vertices **(1000, 1000)** and **(-1000, 5000)**.*

**Radar ACK (Success):**

```
FD FC FB FA 04 00 C2 01 00 00 04 03 02 01
```

---

## 2.3 Radar Data Output Protocol

The LD2450 module communicates with external systems via the serial port and outputs detected target information. This information includes each target's x-coordinate, y-coordinate, and speed. The default serial port settings are 256000 baud rate, 1 stop bit, and no parity.

**Data Output Details:**

- **Frame Rate:** 10 frames per second  
- **Frame Structure:**

```
  Frame Header | Frame Data | Frame Tail
```

- **Frame Header:** `AA FF`  
- **Following Bytes:** `03 00` then the information for target 1, target 2, and target 3  
- **Frame Tail:** `55 CC`

#### Table 9: Report Data Frame Format

Each target's information is provided as follows:

#### Table 10: Data Format within a Frame

| Field                   | Description                                                                                                  |
|-------------------------|--------------------------------------------------------------------------------------------------------------|
| **Target X Coordinate** | signed int16 (most significant bit: 1 indicates positive, 0 indicates negative; unit: mm)                     |
| **Target Y Coordinate** | signed int16 (most significant bit: 1 indicates positive, 0 indicates negative; unit: mm)                     |
| **Target Speed**        | signed int16 (most significant bit: 1 for positive speed, 0 for negative; remaining 15 bits represent speed in cm/s) |
| **Distance Resolution** | uint16 (distance gate size; unit: mm)                                                                         |

**Example Data:**

| Frame Header |        Target 1         |        Target 2         |        Target 3         | Frame Tail |
| :----------: | :---------------------: | :---------------------: | :---------------------: | :--------: |
| AA FF 03 00  | 0E 03 B1 86 10 00 40 01 | 00 00 00 00 00 00 00 00 | 00 00 00 00 00 00 00 00 |   55 CC    |

*Explanation of the Example:*

- **Target 1 is detected:**  
  - **X Coordinate:**  
    - Calculation: `0x0E + (0x03 × 256) = 782`  
    - Adjusted: `0 - 782 = -782 mm`
  - **Y Coordinate:**  
    - Calculation: `0xB1 + (0x86 × 256) = 34481`  
    - Adjusted: `34481 - 2^15 = 1713 mm`
  - **Speed:**  
    - Calculation: `0x10 + (0x00 × 256) = 16`  
    - Adjusted: `0 - 16 = -16 cm/s`
  - **Distance Resolution:**  
    - Calculation: `0x40 + (0x01 × 256) = 320 mm`

- **Targets 2 and 3 are not detected;** their corresponding data segments are `0x00`.

---

## 2.4 Radar Command Configuration Method

The configuration process of the LD2450 radar involves two main steps:

1. **Sending the Command:** The upper computer sends a command to the radar.
2. **Receiving the ACK:** The radar replies with an ACK.  
   *If no ACK is received or the ACK indicates a failure, the configuration command is considered unsuccessful.*

**Configuration Procedure:**

- **Step 1:** Send the **Enable Configuration Command**.  
- **Step 2:** Once a successful ACK is received, send the desired configuration command (e.g., to read parameters).  
- **Step 3:** After receiving a successful ACK for the configuration command, send the **End Configuration Command**.  
- **Step 4:** Upon receiving a successful ACK for the end command, the configuration process is complete.

**Example:**  
To read radar configuration parameters, the upper computer should:
1. Send the *Enable Configuration Command*.
2. After a successful ACK, send the *Read Parameters Command*.
3. After a successful ACK, send the *End Configuration Command*.
4. Once the final ACK is received, the complete parameter read process is concluded.

**Figure 3: Radar Command Configuration Process**  
*(Refer to the original document for the flowchart illustration.)*