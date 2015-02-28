# How to set up raspberry pi as an oldies gaming console

Raspberry pis are cool. Old crappy 2D games are cool. Combine them together and you get an awesome tiny retro gaming system that supports multiple consoles and is portable, for about $50. Here's how!  Total set up time is less than an hour.

*Disclaimer: I am writing this less than 24 hours after first starting to fiddle with my rasberry pi and I am not claiming that what I'm doing is necessarily the correct way to do it.  This is just what worked for me and will hopefully work for others. Another note is that you might want to perform some of the setup items in a different order, and that's fine.*

### Ingredients
- Raspberry pi 
- Mini SD card (and SD card reader/adaptor to be able to write to it from a laptop)
- USB controllers (I bought SNES controllers from Amazon)
- USB keyboard
- Monitor and associated cable (I bought a HDMI -> DVI cable to connect to my monitor)
- Power supply (micro USB, like the one most phones use, is good. I can even power my rpi through my laptop's USB port)
- USB WiFi adapter or ethernet cable (except for being able to access the internet on the rpi, this is also an easy way totransfer files between your machine and your rpi)

## Install raspberry pi on SD card 
- [Download RetroPie project SD card image](http://blog.petrockblock.com/retropie/retropie-downloads/download-info/retropie-sd-card-image-for-rpi-version-1/) (this is a Raspbian image that comes loaded with many emulators, a few games, and the EmulationStation GUI0
- Decompress the file (can use 7-zip on Windows) so that you now have a file with extension `.img`
- [Download Win32 Disk Imager](http://sourceforge.net/projects/win32diskimager/)
- Place the SD card in your laptop and check what letter drive was assigned to it
- Open Win32DiskImager, select the image file that you downloaded and the drive that corresponds to the SD card (**important** - make sure you choose the correct drive letter, otherwise you can maybe severely mess up your harddrive data), click on **Write** and wait a few minutes for it to complete. Close the program when it is done.

## Boot up raspberry pi
- Place the SD card into its slot on the rpi, connect the HDMI from the rpi to a monitor, connect a USB keyboard into the rpi, connect 1 or 2 USB controllers to the rpi, connect a USB WiFi adapter if you have one or an ethernet cable if you don't
- Lastly, connect the power supply
- Hopefully you will now see a your rpi booting up on your monitor. Exciting!!

## Set up raspberry pi
After a minute of many commands running in the terminal, you might see a graphical welcome screen telling you if any gamepads are detected.  This is the EmulationStation graphical user interface, and this is where you'll spend most of your future time - it is the entry point to all the games. It will also be used to configure the controllers. But we'll do that later, first let's do some other rpi setup.  

Press `F4` to quit EmulationStation and go back to the terminal.

### Set up internet
The first thing I like to do is set up internet on the rpi, so that I can install tools and use SSH (more on that later) to make the rest of the setup easier. There are also some setup steps that require a connection. If you have an ethernet cable plugged into rpi, you might already have internet - try running `ping google.com` and see if you get a result. If you don't, then use Google on your real machine to find a fix :)  

If you have a WiFi adapter plugged in, follow these instructions to set up WiFi: 
- Run `lsusb` and make sure you see the WiFi adapter in the list of connected devices
- Run `iwlist wlan0 scan` to see a list of the networks around you. You should be able to see your network in one of the `ESSID` fields
- Run `sudo nano /etc/network/interfaces`
- Make sure the file contents are identical to this
```
auto lo

iface lo inet loopback
iface eth0 inet dhcp

allow-hotplug wlan0
iface wlan0 inet manual
wpa-roam /etc/wpa_supplicant/wpa_supplicant.conf
iface default inet dhcp
```  
- Press `Ctrl`+`x`, `Y`, `Enter` to save the changes if you made any edits
- Run `sudo nano /etc/wpa_supplicant/wpa_supplicant.conf`
- Rhe file should have these two lines (if not, add them)
```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
```  
- Add the following lines at the end of the file (replace `<NETWORK NAME>` and `<NETWORK PASSWORD>` with your network's name and password)
```
network={
ssid="<NETWORK NAME>"
psk="<NETWORK PASSWORD>"
}
```
- Press `Ctrl`+`x`, `Y`, `Enter` to save the file
- Run `sudo ifdown wlan0` followed by `sudo ifup wlan0`
- You might see error messages a-la "Operation not permitted" or "Invalid argument". Ignore those. I hate ignoring error messages, but it's actually fine.
- Reboot rpi with `sudo reboot`
- When rpi comes back (again, press F4 to exit EmulationStation and go to the terminal), try `ping google.com` and hopefully you get a response. We have internet!

The above setup will work for most wireless networks, but if you're on a WPA2-Enterprise network (usually in schools/businesses, require you to provide a username+password to connect), then instead of using the `network={...}` from above, you need to replace that with your network's settings. Try googling/asking for the settings you need. For UBC's ubcsecure network, this is what I need to use:
```
network={
 ssid="ubcsecure"
 scan_ssid=1
 key_mgmt=WPA-EAP
 proto=WPA2 WPA
 eap=PEAP
 group=CCMP TKIP
 pairwise=CCMP TKIP
 ca_cert="/etc/ssl/certs/Thawte_Premium_Server_CA.pem"
 identity="username"
 password="password"
 phase1="peaplabel=0"
 phase2="auth=MSCHAPV2"
}
```
I also added `auto wlan0` to the `/etc/network/interfaces` file, but I'm not sure if that's required or not.

### Install vim editor
This is a personal choice, you don't have to do this, but I prefer vim over nano as my text editor so I want to use it from now on when editing files... `sudo apt-get install vim`

sudo raspi-config
- Select `1 Expand Filesystem`, let it run for a second
- Select `Internationalisation Options`
- Inside Internationalisation Options, select `Change Locale` (it takes a few seconds to open up)
- Assuming you're not in the UK, scroll down to the `en_GB` entry that is selected and press `space` to unselect it. Now scroll to `en_CA.UTF-8` (select UTF, not ISO!) and select it. Press `Enter`.
- If you get prompted with a "watning" about changing the default language, DO select `en_CA` instead of `None`. This will take a minute.
- Go back to Internationalisation Options, now select `Change Timezone` and choose yours (for me: `America`, `Vancouver`)
- Go back to Internationalisation Options, now select `Change Keyboard Layout`, then `Generic 105-key (Intl) PC`, then `Other`, then `English (US)`, then `English (US)` again. Now choose the default option for the next 3 dialogs and wait a minute while the settings change.
- Optional: go to `Overclock` and select a higher setting to increase performance
- Go to `Advanced Options` then `Update` - this will update the configuration script (the one that we're currently using) to the latest version
- When it's done, go to `Finish` and reboot
- 

sudo RetroPie-Setup/... -> update retropie setup script, then register controller for retropie emulator
back to setup -> update input configuration of emulationstation, deselect everything **except** number `(C) Configure video, rewind, and keyboard for RetroArch`





.bashrc
alias ll='ls -l'
alias reboot='sudo shutdown -r now'
alias shutdown='sudo shutdown -h now'



/opt/retropie/emulators/retroarch/retroarch.cfg
# dean: exit a game with select + start, save/load with select+A
input_enable_hotkey_btn = "6"
input_exit_emulator_btn = "7"
input_save_state_btn = "2"




/boot/config.txt
https://github.com/dahano/retropie
arm_freq=900
core_freq=250
sdram_freq=450
over_voltage=2
avoid_safe_mode=1




can use USB to transfer roms



controller configs: /opt/retropie/emulators/retroarch/retroarch.cfg and /opt/retropie/configs/all/retroarch.cfg - not sure yet what the difference is


see what emulators are supported/what file extensions they expect for roms: /etc/emulationstation/es_systems.cfg

#### Quick info
- What's my rpi ip? `hostname -I`
- Go to the GUI `startx`
- Start EmulationStation: `emulationstation` (or `exit` from the terminal will also work)
- Default user name/pw: pi/raspberry
- Exit from ES to terminal: F4
- Switch terminal windows: `ctrl`+`alt`+(`F1`-`F6`)
- Exit from game to ES with controller: `select`+`start`
- Save/load inside game: `select`+`X`
