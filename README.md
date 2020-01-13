# Silent boot on Rasbian Lite

## How it works ?
![diagram](https://raw.githubusercontent.com/ToolReaz/rspi-silent-boot/master/diagram.png)

## Reproduce it
### Raspi-config
First you need to enable autologin.
Execute ``sudo raspi-config`` to enter in the menu and select the following values
- Boot Options > Desktop / CLI > Console Autologin
Then exit the menu and reboot.

### Installing required packages
Update your system and install Xorg, OpenBox, Chromium, feh and Plymouth.
```bash
sudo apt-get update
sudo apt-get upgrade
sudo reboot

sudo apt-get install --no-install-recommends xserver-xorg x11-xserver-utils xinit openbox chromium-browser
sudo apt-get install feh plymouth plymouth-themes
```

### Configuring OpenBox
Edit the file ``/etc/xdg/openbox/autostart`` and replace the following content inside
(it assumes you have wallpaper.png located in /home/pi)
```
# Disable any form of screen saver / screen blanking / power management
xset s off
xset s noblank
xset -dpms

# Background
feh --bg-scale /home/pi/wallpaper.png

# Allow quitting the X server with CTRL-ATL-Backspace
setxkbmap -option terminate:ctrl_alt_bksp

# Start Chromium in kiosk mode
sed -i 's/"exited_cleanly":false/"exited_cleanly":true/' ~/.config/chromium/'Local State'
sed -i 's/"exited_cleanly":false/"exited_cleanly":true/; s/"exit_type":"[^"]\+"/"exit_type":"Normal"/' ~/.config/chromium/Default/Preferences
chromium-browser --disable-infobars --kiosk --disable-pinch 'https://google.com'
```

### Enabling splash screen
Edit ``/boot/cmdline.txt`` as root and add the following lines at the end of the line
```
logo.nologo vt.global_cursor_default=0 quiet splash plymouth.ignore-serial-consoles
```

Edit ``/boot/config.txt`` as root and ensure these lines are correctly set (you may need to adapt depending on your type of display)
```
[pi4]
dtoverlay=i2c-rtc,pcf8563
max_framebuffers=2

[all]
#dtoverlay=vc4-fkms-v3d

disable_splash=1
max_usb_current=1
hdmi_group=2
hdmi_mode=87
hdmi_cvt 800 480 60 6 0 0 0
hdmi_drive=1
```

Import your theme in: ``/usr/share/plymouth/themes/your-theme-folder``

List themes: ``plymouth-set-default-theme --list``

Set default theme: ``sudo plymouth-set-default-theme yourtheme``

Edit the file ``/etc/plymouth/plymouthd.conf`` and ensure these lines are present
```
Theme=your-theme
ShowDelay=0
```

### Start xserver automaticaly
Create a script that your user can execute:
```bash
#!/bin/bash
startx -- -nocursor >> /dev/null 2>&1
```

Create the file ``~.bash_profile`` and write the following content inside
```
[[ -z $DISPLAY && $XDG_VTNR -eq 1 ]] && sh your-script.sh
```

### Remove weak/default password warning
```bash
sudo rm /etc/profile.d/sshpwd.sh
```

### Remove last login message
```bash
sudo chattr +i /var/log/lastlog
```

### Remove the message of the day displayed before login
```bash
sudo rm /etc/motd
sudo rm /etc/update-motd.d/*
```

### Remove login prompt
Create the following file
```bash
sudo systemctl edit getty@tty1
```
and add this content
```
[Service]
ExecStart=
ExecStart=-/sbin/agetty --skip-login --nonewline --noissue --autologin <your-username> --noclear %I $TERM
```
then execute this command
```bash
sudo systemctl reenable getty@tty1
```

## Optionals
### Image between boot and desktop
If you have a black screen between plymouth and openbox you can try to use fbi to display an image. (This solution does not work every times)
<br/>
Install fbi: ``sudo apt-get install fbi``
Edit the file ``/etc/rc.local`` and add the following config befor the line ``exit 0``:
```bash
fbi -noverbose /home/pi/you-image.png &
wait $!
```
