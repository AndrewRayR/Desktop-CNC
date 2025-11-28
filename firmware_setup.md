# CNC.js Setup Guide for Raspberry Pi 4 with RATTMMOTOR GRBL Controller

A complete guide to installing and configuring CNC.js on your Raspberry Pi 4 to control your RATTMMOTOR 2-3 Axis GRBL controller board.

---

## Table of Contents

1. [Hardware Overview](#hardware-overview)
2. [Prerequisites](#prerequisites)
3. [Installation Steps](#installation-steps)
   - [Update Your Raspberry Pi](#1-update-your-raspberry-pi)
   - [Install Node.js](#2-install-nodejs)
   - [Install CNC.js](#3-install-cncjs)
   - [Configure USB Permissions](#4-configure-usb-permissions)
   - [Connect the RATTMMOTOR Controller](#5-connect-the-rattmmotor-controller)
   - [Identify the USB Port](#6-identify-the-usb-port)
4. [Running CNC.js](#running-cncjs)
   - [Manual Start](#manual-start)
   - [Configure CNC.js to Start on Boot](#configure-cncjs-to-start-on-boot)
5. [Initial Configuration](#initial-configuration)
   - [Access Web Interface](#1-access-web-interface)
   - [Connect to Controller](#2-connect-to-controller)
   - [Configure GRBL Settings](#3-configure-grbl-settings)
   - [Spindle Configuration](#4-spindle-configuration)
6. [Testing Your Setup](#testing-your-setup)
   - [Jog Test](#1-jog-test)
   - [Spindle Test](#2-spindle-test)
   - [Homing](#3-homing-if-limit-switches-installed)
7. [Troubleshooting](#troubleshooting)
8. [Optional Enhancements](#optional-enhancements)
   - [Webcam Support](#webcam-support)
   - [Install CNC.js Pendant](#install-cncjs-pendant)
   - [Set Static IP](#set-static-ip)
9. [Safety Reminders](#safety-reminders)
10. [Additional Resources](#additional-resources)
11. [Support](#support)

---

## Hardware Overview

**RATTMMOTOR Controller Specifications:**
- 2-3 Axis GRBL-based controller
- 32-bit processor
- 36V DC input
- Supports 300W/500W 48VDC spindle motors
- USB connectivity
- Built-in cooling fan

---

## Prerequisites

- Raspberry Pi 4 (2GB RAM minimum, 4GB+ recommended)
- MicroSD card (16GB minimum, 32GB recommended)
- Raspberry Pi OS (64-bit recommended)
- Stable internet connection
- USB cable for controller connection
- 36V power supply for the RATTMMOTOR board

---

## Installation Steps

### 1. Update Your Raspberry Pi

```bash
sudo apt update
sudo apt upgrade -y
```

### 2. Install Node.js

CNC.js requires Node.js. Install the latest LTS version:

```bash
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt install -y nodejs
```

Verify installation:

```bash
node --version
npm --version
```

### 3. Install CNC.js

Install CNC.js globally:

```bash
sudo npm install -g cncjs
```

This may take several minutes to complete.

### 4. Configure USB Permissions

Add your user to the dialout group to access USB serial ports:

```bash
sudo usermod -a -G dialout $USER
```

Log out and back in for changes to take effect, or reboot:

```bash
sudo reboot
```

### 5. Connect the RATTMMOTOR Controller

1. Power off your Raspberry Pi
2. Connect the RATTMMOTOR board via USB cable
3. Connect the 36V power supply to the RATTMMOTOR board
4. Power on your Raspberry Pi

### 6. Identify the USB Port

Find your controller's device name:

```bash
ls /dev/tty*
```

Look for `/dev/ttyUSB0` or `/dev/ttyACM0` (most common for GRBL controllers)

You can also use:

```bash
dmesg | grep tty
```

---

## Running CNC.js

### Manual Start

Start CNC.js server:

```bash
cncjs
```

The server will start on port 8000 by default. Access the web interface at:
```
http://[your-pi-ip-address]:8000
```

Or from the Pi itself:
```
http://localhost:8000
```

### Configure CNC.js to Start on Boot

Create a systemd service file:

```bash
sudo nano /etc/systemd/system/cncjs.service
```

Add the following content (replace `pi` with your username if different):

```ini
[Unit]
Description=CNC.js Server
After=network.target

[Service]
Type=simple
User=pi
ExecStart=/usr/bin/cncjs
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

Enable and start the service:

```bash
sudo systemctl enable cncjs.service
sudo systemctl start cncjs.service
```

Check service status:

```bash
sudo systemctl status cncjs.service
```

---

## Initial Configuration

### 1. Access Web Interface

Open a browser and navigate to `http://[raspberry-pi-ip]:8000`

### 2. Connect to Controller

1. Click the "Connection" widget in the left panel
2. Select your serial port (e.g., `/dev/ttyUSB0`)
3. Set baud rate to **115200** (standard for 32-bit GRBL)
4. Click "Open" to connect

### 3. Configure GRBL Settings

Once connected, access the console and check your GRBL version:

```
$$
```

This displays all GRBL configuration parameters. Key settings for your RATTMMOTOR board:

**Important Parameters:**
- `$0` - Step pulse time (microseconds)
- `$1` - Step idle delay (milliseconds)
- `$2` - Step pulse invert
- `$3` - Step direction invert
- `$100`, `$101`, `$102` - Steps per mm for X, Y, Z axes
- `$110`, `$111`, `$112` - Max rate (mm/min) for X, Y, Z
- `$120`, `$121`, `$122` - Acceleration (mm/sec²)
- `$130`, `$131`, `$132` - Max travel (mm) for X, Y, Z

Adjust settings using format: `$[parameter]=[value]`

Example:
```
$100=250
```

### 4. Spindle Configuration

For your 300W/500W spindle motor:

- `$30` - Max spindle speed (RPM)
- `$31` - Min spindle speed (RPM)
- `$32` - Laser mode (0 = disabled for spindle use)

Example configuration:
```
$30=24000
$31=0
$32=0
```

---

## Testing Your Setup

### 1. Jog Test

Use the "Axes" widget to jog each axis:
- Start with small movements (1mm)
- Verify correct direction for X, Y, Z
- If inverted, adjust GRBL `$3` parameter

### 2. Spindle Test

In the console, test spindle:
```
M3 S1000  # Start spindle at low speed
M5        # Stop spindle
```

### 3. Homing (if limit switches installed)

```
$H
```

---

## Troubleshooting

**Cannot connect to controller:**
- Check USB cable connection
- Verify correct port with `ls /dev/tty*`
- Try different baud rates (9600, 115200)
- Check if user is in dialout group: `groups`

**CNC.js won't start:**
- Check Node.js installation: `node --version`
- View logs: `journalctl -u cncjs.service -f`
- Try manual start to see errors: `cncjs`

**Spindle not working:**
- Verify 48V power supply is connected
- Check GRBL spindle settings (`$30`, `$31`, `$32`)
- Ensure PWM spindle control is enabled

**Axes moving in wrong direction:**
- Adjust direction invert setting: `$3=X` (where X is binary value)

---

## Optional Enhancements

### Webcam Support

CNC.js includes a webcam widget that allows you to monitor your CNC machine in real-time from the web interface.

#### Supported Cameras
- Most USB webcams (Logitech, Microsoft, etc.)
- Raspberry Pi Camera Module (via ribbon cable)
- Any Linux-compatible camera that creates a `/dev/video0` device

#### Step 1: Verify Camera Detection

Connect your USB webcam and check if it's detected:

```bash
ls /dev/video*
```

You should see `/dev/video0` (or `/dev/video1` if you have multiple cameras).

For additional verification:

```bash
v4l2-ctl --list-devices
```

#### Step 2: Install Dependencies

Install required packages for mjpg-streamer:

```bash
sudo apt install cmake libjpeg-dev gcc g++ libv4l-dev
```

#### Step 3: Build and Install mjpg-streamer

Download and compile mjpg-streamer:

```bash
cd ~
git clone https://github.com/jacksonliam/mjpg-streamer.git
cd mjpg-streamer/mjpg-streamer-experimental
make
sudo make install
```

#### Step 4: Test Your Camera

Test the camera stream manually (adjust resolution if needed):

```bash
mjpg_streamer -i "input_uvc.so -d /dev/video0 -r 1280x720 -f 30" -o "output_http.so -p 8080 -w /usr/local/share/mjpg-streamer/www"
```

Open a browser and navigate to:
```
http://[raspberry-pi-ip]:8080/?action=stream
```

You should see your camera feed. Press `Ctrl+C` to stop the test.

**Note:** If you get an error about resolution, try lower resolutions like `640x480` or `800x600`.

#### Step 5: Create Webcam Service

Create a systemd service for automatic startup:

```bash
sudo nano /etc/systemd/system/webcam.service
```

Add the following content (adjust resolution as needed):

```ini
[Unit]
Description=MJPG Streamer for CNC Webcam
After=network.target

[Service]
Type=simple
User=pi
ExecStart=/usr/local/bin/mjpg_streamer -i "input_uvc.so -d /dev/video0 -r 1280x720 -f 15" -o "output_http.so -p 8080 -w /usr/local/share/mjpg-streamer/www"
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

**Resolution Options:**
- `640x480` - Low quality, minimal CPU usage
- `1280x720` - HD quality, moderate CPU usage (recommended)
- `1920x1080` - Full HD, higher CPU usage

Enable and start the service:

```bash
sudo systemctl enable webcam.service
sudo systemctl start webcam.service
```

Check service status:

```bash
sudo systemctl status webcam.service
```

#### Step 6: Configure CNC.js Webcam Widget

1. Open CNC.js web interface
2. Click the gear icon (⚙️) in the top right
3. Select "Widgets" from the left menu
4. Enable the "Webcam" widget if not already visible
5. In the webcam widget, click the settings icon
6. Select "Use a M-JPEG stream over HTTP"
7. Enter the stream URL:
   ```
   http://localhost:8080/?action=stream
   ```
   Or use your Pi's IP address if accessing from another device:
   ```
   http://[raspberry-pi-ip]:8080/?action=stream
   ```
8. Click "OK" to save

Your webcam feed should now appear in the CNC.js interface!

#### Troubleshooting Webcam Issues

**Black screen in webcam widget:**
- Verify stream URL is correct
- Check if mjpg-streamer service is running: `sudo systemctl status webcam.service`
- Test stream directly in browser: `http://[pi-ip]:8080/?action=stream`
- Try a lower resolution in the service file

**Camera not detected:**
- Check USB connection
- Verify camera appears: `ls /dev/video*`
- Try different USB port
- Some cameras require additional drivers

**Poor performance/lag:**
- Reduce resolution in service file (e.g., `640x480`)
- Lower frame rate (e.g., `-f 10` instead of `-f 15`)
- Check network connection if viewing remotely

**Using Raspberry Pi Camera Module:**

If using the Pi Camera via ribbon cable instead of USB:

```bash
sudo raspi-config
```

Navigate to "Interface Options" → "Camera" → Enable

Modify the webcam service to use the raspicam input:

```ini
ExecStart=/usr/local/bin/mjpg_streamer -i "input_raspicam.so -fps 15 -x 1280 -y 720" -o "output_http.so -p 8080 -w /usr/local/share/mjpg-streamer/www"
```

#### Optional: Recording

You can record your CNC jobs by capturing the stream with ffmpeg:

```bash
sudo apt install ffmpeg
```

Record manually:

```bash
ffmpeg -i "http://localhost:8080/?action=stream" -c copy ~/cnc-recording-$(date +%Y%m%d-%H%M%S).mjpeg
```

---

### Install CNC.js Pendant

For mobile/tablet control:

```bash
sudo npm install -g cncjs-pendant-tinyweb
```

---

### Set Static IP

Edit dhcpcd configuration:

```bash
sudo nano /etc/dhcpcd.conf
```

Add (adjust to your network):
```
interface eth0
static ip_address=192.168.1.100/24
static routers=192.168.1.1
static domain_name_servers=192.168.1.1
```

---

## Safety Reminders

- Always home your machine before starting a job
- Set proper work zero before cutting
- Keep emergency stop accessible
- Monitor first jobs closely
- Ensure proper ventilation when cutting
- Wear safety glasses

---

## Additional Resources

- [CNC.js Documentation](https://github.com/cncjs/cncjs)
- [GRBL Configuration Guide](https://github.com/gnea/grbl/wiki)
- [CNC.js Community](https://github.com/cncjs/cncjs/discussions)

---

## Support

If you encounter issues:
1. Check CNC.js GitHub issues
2. Review GRBL configuration
3. Verify hardware connections
4. Check system logs: `journalctl -xe`

---

**Note:** Always refer to your RATTMMOTOR controller's specific documentation for wiring diagrams, pinouts, and electrical specifications before connecting motors and limit switches.

---
