# Digital Fence Controller

A touchscreen-based controller for CNC machine fences running on Raspberry Pi. This application provides precise control over fence positioning using a stepper motor, offering both metric and imperial measurements.

![Digital Fence Controller UI](https://i.imgur.com/placeholder.jpg)

## Features

- **Touchscreen Optimized Interface**: Designed for 10.1" resistive touchscreens
- **Dual Measurement System**: Switch between metric (mm) and imperial (inches)
- **Home Position Management**: Set and recall absolute and relative home positions
- **Precise Positioning**: Digital readout with 0.001 unit precision
- **Nudge Controls**: Quick incremental movements with configurable distances
- **Emergency Stop**: Immediate motor halt for safety
- **Power Control**: Safely shut down the system from the interface
- **Auto-start**: Application launches automatically on boot
- **No Cursor**: Cursor is hidden for a cleaner touchscreen experience
- **No Screensaver**: Screen stays on permanently for continuous operation

## Hardware Requirements

- Raspberry Pi 4 (2GB+ RAM recommended)
- 10.1" IPS Resistive Touchscreen LCD (1024×600)
- Nema23 Stepper Motor (60BYGH301B or similar)
- CW5045 Stepper Motor Driver
- Mean Well RSP-200-48 Power Supply (or similar 48V supply)
- MOD1 Rack and Pinion system

## Software Requirements

- Raspberry Pi OS Lite (64-bit)
- Node.js 18.x
- Electron
- React

## Installation

### Automatic Installation

The easiest way to install is using the provided setup script:

1. Start with a fresh installation of Raspberry Pi OS Lite
2. Clone this repository or download the setup script:
   ```bash
   wget https://raw.githubusercontent.com/yourusername/digital-fence/main/setup.sh
   ```

3. Make the script executable:
   ```bash
   chmod +x setup.sh
   ```

4. Run the script with sudo:
   ```bash
   sudo ./setup.sh
   ```

5. Follow the prompts and reboot when finished

### Manual Installation

If you prefer to install manually, follow these steps:

1. Install the required packages:
   ```bash
   sudo apt update
   sudo apt install -y git curl wget build-essential xserver-xorg x11-xserver-utils xinit lightdm xinput evtest unclutter
   sudo apt install -y libgtk-3-dev libgconf-3-0
   ```

2. Install Node.js:
   ```bash
   curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
   sudo apt install -y nodejs
   ```

3. Create a user for the application:
   ```bash
   sudo useradd -m -G sudo,video,input,tty -s /bin/bash digifence
   sudo passwd digifence
   ```

4. Configure auto-login in LightDM:
   ```bash
   sudo mkdir -p /etc/lightdm/lightdm.conf.d/
   echo -e "[Seat:*]\nautologin-user=digifence\nautologin-user-timeout=0\nxserver-command=X -s 0 -dpms -nocursor" | sudo tee /etc/lightdm/lightdm.conf
   ```

5. Configure touchscreen calibration:
   ```bash
   sudo mkdir -p /etc/X11/xorg.conf.d/
   echo -e 'Section "InputClass"\n    Identifier      "Touchscreen Calibration"\n    MatchProduct    "ADS7846 Touchscreen"\n    MatchDevicePath "/dev/input/event4"\n    Driver          "libinput"\n    Option          "CalibrationMatrix" "1 0 0 0 1 0 0 0 1"\n    Option          "TransformationMatrix" "1 0 0 0 1 0 0 0 1"\nEndSection' | sudo tee /etc/X11/xorg.conf.d/99-calibration.conf
   ```

6. Disable cursor and screensaver:
   ```bash
   echo '#!/bin/bash\nexport DISPLAY=:0\nxset s off\nxset -dpms\nxset s noblank\nunclutter -idle 0 &' > ~/disable-screensaver-cursor.sh
   chmod +x ~/disable-screensaver-cursor.sh
   ```

7. Create autostart entry:
   ```bash
   mkdir -p ~/.config/autostart
   echo -e '[Desktop Entry]\nType=Application\nName=Disable Screensaver and Cursor\nExec=/home/digifence/disable-screensaver-cursor.sh\nTerminal=false\nX-GNOME-Autostart-enabled=true' > ~/.config/autostart/disable-screensaver-cursor.desktop
   ```

8. Clone the project repository and install dependencies:
   ```bash
   cd ~
   git clone https://github.com/yourusername/digital-fence.git
   cd digital-fence
   npm install
   npm run build
   ```

9. Create autostart entry for the application:
   ```bash
   echo -e '[Desktop Entry]\nType=Application\nName=Digital Fence Controller\nExec=/home/digifence/digital-fence/start.sh\nTerminal=false\nX-GNOME-Autostart-enabled=true' > ~/.config/autostart/digital-fence.desktop
   echo -e '#!/bin/bash\ncd /home/digifence/digital-fence\nnpm start' > ~/digital-fence/start.sh
   chmod +x ~/digital-fence/start.sh
   ```

10. Reboot the system:
    ```bash
    sudo reboot
    ```

## Touchscreen Calibration

If your touchscreen needs calibration:

1. Install the calibration tool:
   ```bash
   sudo apt install -y xinput-calibrator
   ```

2. Stop the application and run the calibration:
   ```bash
   # Find and kill the application process
   ps aux | grep electron
   sudo kill <PID>
   
   # Create a calibration script
   cat > ~/calibrate-touch.sh << 'EOF'
   #!/bin/bash
   export DISPLAY=:0
   xinput_calibrator --output-type xorg.conf.d > ~/calibration-output.txt
   EOF
   
   chmod +x ~/calibrate-touch.sh
   
   # Run the calibration
   sudo ./calibrate-touch.sh
   ```

3. Follow the on-screen instructions, then update your configuration:
   ```bash
   sudo cp ~/calibration-output.txt /etc/X11/xorg.conf.d/99-calibration.conf
   ```

4. Restart the system:
   ```bash
   sudo reboot
   ```

## Hardware Integration

The application includes a stub for stepper motor control. To integrate with your physical hardware:

1. Update the `StepperControl.js` file with your GPIO pin configurations:
   ```javascript
   this.stepPin = new this.Gpio(17, {mode: this.Gpio.OUTPUT});
   this.dirPin = new this.Gpio(27, {mode: this.Gpio.OUTPUT});
   this.enablePin = new this.Gpio(22, {mode: this.Gpio.OUTPUT});
   ```

2. Adjust the `STEPS_PER_MM` constant in `App.js` based on your mechanical setup:
   ```javascript
   const STEPS_PER_MM = 35.37; // For MOD1 rack with 18T pinion at 1/10 microstepping
   ```

## Usage

The interface is divided into three main sections:

### Left Section
- **Emergency Stop Button**: Immediately halts all movement
- **Units Toggle**: Switch between mm and inches
- **Current Position Display**: Shows position relative to home
- **Target Position Display**: Shows destination for next movement
- **Speed Control**: Slider to adjust movement speed

### Center Section
- **Absolute Home Controls**: Set and go to absolute reference point
- **Relative Home Controls**: Set and go to relative reference point
- **Nudge Value Selectors**: Choose increment distances
- **Power Off Button**: Safely shut down the system

### Right Section
- **Nudge Buttons**: Move left or right by selected increment
- **Numeric Keypad**: Enter precise target positions
- **Enter Button**: Confirm movement to entered position

## Troubleshooting

### Screen appears blank on startup
- Check the display connection
- Verify the X server is running: `ps aux | grep X`
- Check logs: `sudo journalctl -xe`

### Touch input not working correctly
- Run the calibration procedure described above
- Check touchscreen connection
- Verify the device is recognized: `ls -l /dev/input/event*`

### Application doesn't start automatically
- Check autostart entries: `ls -la ~/.config/autostart/`
- Verify permissions: `ls -la ~/digital-fence/start.sh`
- Check application logs: `journalctl -u lightdm`

### Motor doesn't move
- Check wiring connections
- Verify driver DIP switch settings
- Test GPIO pins manually:
  ```bash
  sudo apt install -y pigpio
  sudo pigpiod
  pigs w 17 1  # Test step pin
  pigs w 17 0
  ```

## License

[MIT](LICENSE)

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## Acknowledgments

- Developed for woodworking CNC applications
- Uses MOD1 rack and pinion system for precision movement
- Built with React and Electron for a responsive UI
