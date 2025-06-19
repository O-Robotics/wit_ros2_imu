# IMU with ROS 2 on Raspberry Pi

This guide walks you through setting up a USB-connected IMU module (based on CH340/CH341 serial chip) on a **Raspberry Pi CM4 / Pi 4** running ROS 2 (Humble).

The device will be accessible via a symbolic link `/dev/imu_usb` for consistent access.

---

## Hardware Requirements

- Raspberry Pi CM4 or Pi 4 with USB ports
- IMU module using CH340/CH341 USB-to-Serial chip
- ROS 2 Humble installed on the Pi

---

## Step 1: Verify IMU Connection

1. Plug in the IMU and check USB devices:

```bash
lsusb
```

Look for output like:

```
Bus 001 Device 006: ID 1a86:7523 QinHeng Electronics CH340 serial converter
```

2. Check for assigned serial devices:

```bash
ls /dev/ttyUSB*
```

e.g., `/dev/ttyUSB0`, `/dev/ttyUSB5`, etc.

3. Confirm the kernel driver detected the device:

```bash
dmesg | grep ch34
```

Expected output:

```
ch341-uart converter now attached to ttyUSB5
```

Rpi have this driver, while Jetson nano orin need to install the driver and compile.

## Step 2: Set up udev Rule to Create /dev/imu_usb

You can configure the rule either manually or via a script.

### Method 1: Manual

```bash
sudo nano /etc/udev/rules.d/imu_usb.rules
```

Add the following line:

```
KERNEL=="ttyUSB*", ATTRS{idVendor}=="1a86", ATTRS{idProduct}=="7523", MODE:="0777", SYMLINK+="imu_usb"
```

#### Apply the rule:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger
```

### Method 2: Script

Run the provided shell script (make sure it's executable):

```bash
chmod +x bind_usb.sh
./bind_usb.sh
```

This will copy `imu_usb.rules` to `/etc/udev/rules.d/`.

### Verify:

```bash
ls -l /dev/imu_usb
```

You should see a symlink like:

```
/dev/imu_usb -> ttyUSB5
```

---

## Step 3: Test Serial Output (Optional)

To verify IMU is sending data:

```bash
sudo apt install screen
screen /dev/imu_usb 9600
```

To exit: `Ctrl + A`, then `K`, then press `Y`.

---

## Step 4: Build ROS 2 IMU package

```bash
cd ~/ros2_ws/src
git clone https://github.com/O-Robotics/wit_ros2_imu.git
cd ..
colcon build --packages-select wit_ros2_imu
```

---

## Step 5: Launch ROS 2 IMU Node

Assuming you named the package `wit_ros2_imu`, otherwise, change to your own:

```
source ~/Documents/localization_ws/install/setup.bash
```

```bash
ros2 launch wit_ros2_imu rviz_and_imu.launch.py
```

---

##  Step 5: Verify ROS 2 Topic

```bash
ros2 topic list
ros2 topic echo /imu/data_raw
```

You should see the IMU data publishing.

---

##  Notes

- Default baud rate: 9600 (match your device setting)
- If using multiple serial devices, use additional identifiers like `serial` in your udev rule

---

##  Troubleshooting

| Issue                    | Suggestion                                    |
| ------------------------ | --------------------------------------------- |
| `/dev/ttyUSB*` not found | Check cable, power, and device recognition    |
| `/dev/imu_usb` missing   | Re-check udev rule, reload, unplug/replug     |
| No ROS topic output      | Verify baud rate, topic name, and device path |
