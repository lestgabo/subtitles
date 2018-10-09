# Android Things Practice
by Lestley Gabo

Using Android Things with Raspberry Pi 3b

***************************************************************************
***************************************************************************
Using MPI3508 LCD touchscreen for Raspberry Pi ordered from Amazon.ca
***************************************************************************
STEPS FOLLOWED TO MAKE LCD TOUCHSCREEN FUNCTION (IMPORTANT):
***************************************************************************
***************************************************************************

For Headless mode:
  This link was helpful:
  http://www.circuitbasics.com/access-raspberry-pi-desktop-remote-connection/
  Turn on vnc inside raspberry pi, the latest version has it built in!

    open CMD:
    ```bash
      sudo raspi-config
      (Interfacing Options > VNC > Enable > Yes)
      (if getting lines that say can't find things, you need to update && upgrade first)
    ```

  Download RealVNC on desktop computer! (self explanatory)
    Get the IP address from pi in order to remote into it.
    As per the link above, an easy way to find raspi IP is to use Advanced IP Scanner.

***************************************************************************

For turning on LCD:

  open CMD:
    ```bash
    mkdir lcd
    cd lcd
      git clone https://github.com/goodtft/LCD-show
      (asked for git credentials, you need a git account)
      chmod -R 755 LCD-show
      cd LCD-show/
        sudo ./MPI3508_480_320-show
    ```
***************************************************************************

For static IP:
  This link was helpful:
  https://raspberrypi.stackexchange.com/questions/37920/how-do-i-set-up-networking-wifi-static-ip-address/74428#74428
  dhcpcd method

    open CMD:
      ```bash
      (get IP address, need that /# after the ip address)
      ip -4 addr show | grep global

      (get default IP address of router)
      ip route | grep default | awk '{print $3}'
      ```
    Edit /etc/dhcpcd.conf as follows:

      open CMD:
      ```bash
        sudo nano /etc/dhcpcd.conf
          (I put this in the very top)
          interface wlan0
          static ip_address=192.168.x.x/x
          static routers=192.168.x.x
          static domain_name_servers=192.168.x.x
        ```

***************************************************************************

For LCD calibration:
  This link was helpful:
    http://www.circuitbasics.com/raspberry-pi-touchscreen-calibration-screen-rotation/
  This updates the calibration tools used, I think.

  open CMD:
    ```bash
    sudo apt-get install xserver-xorg-input-evdev
    sudo cp -rf /usr/share/X11/xorg.conf.d/10-evdev.conf /usr/share/X11/xorg.conf.d/45-evdev.conf
    sudo reboot
    ```

  This installs Xinput for touchscreen calibration and uses it.

    open CMD:
      ```bash
      cd LCD-show/
        sudo dpkg -i -B xinput-calibrator_0.7.5-1_armhf.deb
      ```

        (click on the 4 corners to get inputs, e.g. 341 65331 1004 65858)
        ```bash
        DISPLAY=:0.0 xinput_calibrator
        ```

        (copy inputs here)
        ```bash
        sudo nano /etc/X11/xorg.conf.d/99-calibration.conf
        ```

***************************************************************************

For inverted inputs:

  This link was helpful:
  http://www.circuitbasics.com/raspberry-pi-touchscreen-calibration-screen-rotation/

  I already followed the LCD calibration above.
  My calibration was correct, but had an issue of touchscreen input going up when stylus goes down and vice versa.
  This also happens when I go left on the stylus, the input goes right and vice versa.

  So I did some troubleshooting:

    troubleshooting #1:
      For some reason there was a lot of copies of Section "InputClass" when I went to
      sudo nano /usr/share/X11/xorg.conf.d/10-evdev.conf, so I deleted the copies
      Then I added --> Option "SwapAxes" "1" and Option "InvertY" "1"

      open CMD:
      ```bash
        sudo nano /usr/share/X11/xorg.conf.d/10-evdev.conf
          Section "InputClass"
            Identifier "evdev pointer catchall"
            MatchIsPointer "on"
            MatchDevicePath "/dev/input/event*"
            Driver "evdev"
            Option "SwapAxes" "1"
            Option "InvertY" "1"
          EndSection
      ```

    troubleshooting #2:

      I actually plugged in the power into the LCD touchscreen itself, never even noticed.
      I don't think this made a difference, but I unplugged it then put the power into the raspi instead.

    troubleshooting #3:

      Redid the instructions to turn on the LCD.

      Turned back to HDMI mode:

        open CMD:
        ```bash
          cd lcd/LCD-show
            chmod 777 LCD-hdmi
            sudo ./LCD-hdmi
            (reboots by itself from the script)
        ```
      Turned on LCD mode:

      ```bash
        open CMD:
          cd lcd/LCD-show
            sudo ./MPI3508_480_320-show
      ```
  I got lucky? And the inverted inputs were fixed after doing the 3 troubleshooting steps.


*********************************************************
