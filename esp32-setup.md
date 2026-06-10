# ESP32 Attendance System - Setup Guide

## Overview
This guide explains how to connect an ESP32 microcontroller to the attendance system with static IP configuration for reliable network communication.

---

## Hardware Requirements

- **ESP32 Development Board** (ESP32-WROOM-32 or similar)
- **USB Cable** (Type-A to Micro-USB)
- **WiFi Network** (2.4 GHz - ESP32 doesn't support 5 GHz)
- **Computer** with Arduino IDE installed
- **Optional**: USB Camera module (for photo capture)

---

## Software Setup

### Step 1: Install Arduino IDE
1. Download from: https://www.arduino.cc/en/software
2. Install on your computer

### Step 2: Add ESP32 Board Support
1. Open Arduino IDE
2. Go to **File → Preferences**
3. Add this URL in "Additional Board Manager URLs":
   ```
   https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
   ```
4. Go to **Tools → Board → Board Manager**
5. Search for "esp32" and install **esp32 by Espressif Systems**

### Step 3: Install Required Libraries
In Arduino IDE, go to **Sketch → Include Library → Manage Libraries** and install:
- **WebServer** (built-in)
- **WiFi** (built-in)
- **EEPROM** (built-in)
- **ArduinoJson** (by Benoit Blanchon)
- **AsyncWebServer** (by lacamera) - optional for async operations

---

## Network Configuration

### Static IP Setup

The ESP32 needs to be configured with a static IP address for reliable communication. Use these parameters:

```
Static IP:    192.168.1.100
Gateway:      192.168.1.1
Subnet Mask:  255.255.255.0
DNS Server:   8.8.8.8 (or your router's IP)
```

**Note:** Adjust the IP range based on your WiFi network. Check your router settings to find the appropriate subnet.

### Steps to Configure in Admin Panel

1. Login to Admin Portal
2. Go to **ESP32 Config** section
3. Enter these details:
   - **Static IP Address**: `192.168.1.100` (choose an unused IP)
   - **Gateway**: `192.168.1.1` (your router's IP)
   - **Subnet Mask**: `255.255.255.0`
   - **DNS Server**: `8.8.8.8` or `1.1.1.1`
   - **WiFi SSID**: Your network name
   - **WiFi Password**: Your network password
   - **Server Port**: `8080` (default)
   - **Device ID**: `ESP32-001` (give it a unique name)

4. Click **Save Configuration**
5. Click **Download Firmware Code** to get the Arduino sketch

---

## Uploading Firmware

### Step 1: Prepare the Sketch
1. Download the firmware code from the admin panel
2. Open Arduino IDE
3. Paste the firmware code into a new sketch
4. Save it as `attendance_esp32.ino`

### Step 2: Select Board and Port
1. Go to **Tools → Board** and select **ESP32 Dev Module**
2. Go to **Tools → Port** and select the COM port (e.g., COM3)
3. Set **Upload Speed** to `921600` or `115200`
4. Set **Flash Frequency** to `80 MHz`

### Step 3: Upload
1. Connect ESP32 via USB
2. Click **Upload** (→ button) in Arduino IDE
3. Wait for "Hard resetting via RTS pin..." message
4. Your code is now on the ESP32

### Step 4: Verify Connection
1. Open **Tools → Serial Monitor**
2. Set baud rate to `115200`
3. Press the **Reset** button on ESP32
4. You should see output like:
   ```
   [ESP32] Attendance System Starting...
   [ESP32] Device ID: ESP32-001
   [WiFi] Connected!
   [WiFi] IP: 192.168.1.100
   [Server] Started on port 8080
   ```

---

## API Endpoints

Once the ESP32 is connected, it exposes these HTTP endpoints:

### 1. **Status Check**
```bash
GET http://192.168.1.100:8080/status
```
Returns:
```json
{
  "device_id": "ESP32-001",
  "ip": "192.168.1.100",
  "status": "online",
  "signal": -45,
  "uptime": 12345
}
```

### 2. **Attendance Record**
```bash
POST http://192.168.1.100:8080/attendance
Content-Type: application/json

{
  "employee_id": "EMP001",
  "name": "John Doe",
  "type": "In",
  "time": "09:30",
  "timestamp": "2026-06-10T09:30:00Z"
}
```

### 3. **Configuration**
```bash
GET http://192.168.1.100:8080/config
```
Returns device and network configuration

### 4. **Photo Upload**
```bash
POST http://192.168.1.100:8080/photo
Content-Type: application/octet-stream

[Binary image data]
```

### 5. **Ping (Heartbeat)**
```bash
GET http://192.168.1.100:8080/ping
```
Returns: `pong`

---

## Testing the Connection

### From Admin Panel:
1. Go to **ESP32 Config**
2. Enter the Static IP and Port
3. Click **Test Connection**
4. You should see: ✓ Connection successful!

### From Command Line (Windows PowerShell):
```powershell
# Test status endpoint
Invoke-WebRequest -Uri "http://192.168.1.100:8080/status" -Method GET

# Test ping
Invoke-WebRequest -Uri "http://192.168.1.100:8080/ping" -Method GET
```

### From Linux/Mac Terminal:
```bash
# Test status
curl -X GET http://192.168.1.100:8080/status

# Test ping
curl -X GET http://192.168.1.100:8080/ping
```

---

## Troubleshooting

### WiFi Not Connecting
- Check SSID and password are correct
- Ensure ESP32 is within range of router
- Verify router is 2.4 GHz (not 5 GHz)
- Check serial output for errors

### Static IP Not Working
- Ensure IP is not already in use
- Verify gateway IP matches your router
- Check subnet mask is correct
- Try power cycling the ESP32

### Cannot Reach Server
- Verify ESP32 and computer are on same network
- Check firewall is not blocking port 8080
- Test with different port if port 8080 is blocked
- Ensure ESP32 shows "Connected!" in serial monitor

### Serial Monitor Shows Garbage
- Change baud rate to 115200
- Check USB cable connection
- Try a different USB port
- Reinstall ESP32 board drivers

---

## Advanced Configuration

### Change Port Number
Edit this line in the firmware:
```cpp
const int PORT = 8080;  // Change to 8081, 9000, etc.
```

### Change Device ID
Edit this line:
```cpp
const char* DEVICE_ID = "ESP32-001";  // Change as needed
```

### Add Authentication
To add username/password protection:
```cpp
const char* AUTH_USER = "admin";
const char* AUTH_PASS = "password123";

// In handler function, add:
if (server.hasArg("user") && server.hasArg("pass")) {
  if (server.arg("user") == AUTH_USER && server.arg("pass") == AUTH_PASS) {
    // Process request
  }
}
```

### Enable HTTPS/SSL
Add **WiFiClientSecure** and SSL certificates for encrypted communication.

---

## Network Diagram

```
                    WiFi Router (192.168.1.1)
                            |
                 ___________|___________
                |                       |
          [Your Computer]         [ESP32 Device]
          192.168.1.x             192.168.1.100:8080
                |                       |
          Admin Portal          Attendance Server
          employee.html         (WebServer running)
          admin.html
```

---

## Security Recommendations

1. **Change default credentials** in admin panel
2. **Use strong WiFi password** (minimum 12 characters)
3. **Change device ID** to something unique
4. **Keep ESP32 updated** with latest firmware
5. **Use HTTPS** in production (with SSL certificates)
6. **Restrict network access** using firewall rules
7. **Regular backups** of attendance data
8. **Update passwords** regularly

---

## Performance Tips

- **Battery**: ESP32 with WiFi consumes ~80-100mA - use power adapter
- **Latency**: Static IP ensures faster connection than DHCP
- **Range**: Place ESP32 closer to router for better signal strength
- **Thermal**: Ensure adequate ventilation around ESP32
- **Reliability**: Use quality USB cable and proper power supply

---

## Support & References

- **ESP32 Documentation**: https://docs.espressif.com/projects/esp-idf/
- **Arduino-ESP32 GitHub**: https://github.com/espressif/arduino-esp32
- **WebServer Library**: https://github.com/espressif/arduino-esp32/tree/master/libraries/WebServer

---

## Version Information

- **System Version**: 1.0
- **ESP32 Board**: ESP32 Dev Module
- **Framework**: Arduino
- **Last Updated**: June 2026

---

*For issues or questions, refer to the admin panel documentation or contact support.*
