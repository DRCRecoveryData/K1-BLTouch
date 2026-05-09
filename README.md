# Creality K1 - Standard Klipper Transition & BLTouch Integration

This repository contains the configuration notes, shell scripts, and documentation for converting a Creality K1 to a "Pure" Klipper environment. This setup replaces the proprietary Creality modules with official Klipper modules to ensure better compatibility and stability.

## 🚀 Overview

The goal of this modification is to:
*   Replace the stock load-cell leveling (`prtouch_v2`) with a **BLTouch/CR-Touch**.
*   Remove proprietary Creality bloatware (`fan_feedback`, `custom_macro`).
*   Restore standard Klipper behavior for easier maintenance and updates.

## 🛠 Hardware Changes

| Component | Stock Configuration | Upgraded Configuration |
| :--- | :--- | :--- |
| **Z-Endstop** | Load Cell (PRTouch) | **BLTouch / Virtual Endstop** |
| **Firmware** | Creality OS (Modified) | **Standard Klipper Modules** |

## 💻 Installation & Scripting

### 1. Module Replacement
Run these commands via SSH to swap Creality's modified Python files with official Klipper releases and remove proprietary modules:

```sh
# Backup existing config
mkdir -p /usr/data/klipper_backup
cp -r /usr/share/klipper /usr/data/klipper_backup/
cp -r /usr/data/printer_data/config /usr/data/klipper_backup/

# Replace core modules
cd /usr/share/klipper/klippy/extras
curl -O [https://raw.githubusercontent.com/Klipper3d/klipper/master/klippy/extras/bltouch.py](https://raw.githubusercontent.com/Klipper3d/klipper/master/klippy/extras/bltouch.py)
curl -O [https://raw.githubusercontent.com/Klipper3d/klipper/master/klippy/extras/homing.py](https://raw.githubusercontent.com/Klipper3d/klipper/master/klippy/extras/homing.py)
curl -O [https://raw.githubusercontent.com/Klipper3d/klipper/master/klippy/extras/probe.py](https://raw.githubusercontent.com/Klipper3d/klipper/master/klippy/extras/probe.py)
rm -f bltouch.pyc homing.pyc probe.pyc

# Remove Creality bloatware
rm -f fan_feedback.py fan_feedback.pyc prtouch_v2.py prtouch_v2.pyc custom_macro.py custom_macro.pyc

```

### 2. Automatic `printer.cfg` Cleanup

The following automation removes conflicting sections and updates the Z-endstop logic:

```sh
CONFIG_PATH="/usr/data/printer_data/config/printer.cfg"

# Remove proprietary sections
sed -i '/^\[prtouch_v2\]/,/^\[/ { /^\[prtouch_v2\]/d; /^\[/!d }' $CONFIG_PATH
sed -i '/^\[fan_feedback\]/,/^\[/ { /^\[fan_feedback\]/d; /^\[/!d }' $CONFIG_PATH

# Update Stepper Z for BLTouch
sed -i 's/^position_endstop:/# position_endstop:/g' $CONFIG_PATH
sed -i 's/^endstop_pin: .*/endstop_pin: probe:z_virtual_endstop/g' $CONFIG_PATH

# Disable legacy Creality macros
sed -i 's/CX_PRINT_DRAW_ONE_LINE/# CX_PRINT_DRAW_ONE_LINE/g' $CONFIG_PATH
sed -i 's/CX_NOZZLE_CLEAR/# CX_NOZZLE_CLEAR/g' $CONFIG_PATH

```

## ⚙️ Key Configuration Settings

### BLTouch Integration

Add the following to your `printer.cfg` (ensure pin mappings match your hardware):

```yaml
[bltouch]
control_pin: nozzle_mcu:PA8
sensor_pin: ^nozzle_mcu:PA9
x_offset: -38
y_offset: -5
z_offset: 0 # CALIBRATE MANUALLY!
speed: 5
lift_speed: 10

```

## ⚠️ Safety Disclaimer

> [!WARNING]
> Removing `prtouch_v2` disables the stock nozzle-probing safety mechanism. Ensure your BLTouch is correctly deployed before homing Z to prevent the nozzle from crashing into the bed.

## 📜 License

This project is for educational purposes. Use at your own risk.
