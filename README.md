# How to set up raspberry pi as an oldies gaming console

Raspberry pis are cool. Old crappy 2D games are cool. Combine them together and you get an awesome tiny retro gaming system that supports multiple consoles and is portable, for about $50. Here's how!  Total set up time is less than an hour. I am able to run all sorts of games ranging from Super Mario World on SNES to Crash Bandicoot on PS1.

*Disclaimer: I am writing this less than 24 hours after first starting to fiddle with my rasberry pi and I am not claiming that what I'm doing is necessarily the correct way to do it.  This is just what worked for me and will hopefully work for others. Another note is that you might want to perform some of the setup items in a different order, and that's fine.*

### Ingredients
- Raspberry pi 
- Mini SD card (and SD card reader/adaptor to be able to write to it from a laptop)
- USB controllers (I bought SNES controllers from Amazon)
- USB keyboard
- Monitor and associated cable (I bought a HDMI -> DVI cable to connect to my monitor)
- Power supply (micro USB, like the one most phones use, is good. I can even power my rpi through my laptop's USB port)
- USB WiFi adapter or ethernet cable (except for being able to access the internet on the rpi, this is also an easy way totransfer files between your machine and your rpi)

### Install raspberry pi on SD card 
- [Download RetroPie project SD card image](http://blog.petrockblock.com/retropie/retropie-downloads/download-info/retropie-sd-card-image-for-rpi-version-1/) (this is a Raspbian image that comes loaded with many emulators, a few games, and the EmulationStation GUI0
- Decompress the file (can use 7-zip on Windows) so that you now have a file with extension `.img`
- [Download Win32 Disk Imager](http://sourceforge.net/projects/win32diskimager/)
- Place the SD card in your laptop and check what letter drive was assigned to it
- Open Win32DiskImager, select the image file that you downloaded and the drive that corresponds to the SD card (**important** - make sure you choose the correct drive letter, otherwise you can maybe severely mess up your harddrive data), click on **Write** and wait a few minutes for it to complete. Close the program when it is done.

### Boot up raspberry pi
- Place the SD card into its slot on the rpi, connect the HDMI from the rpi to a monitor, connect a USB keyboard into the rpi, connect 1 or 2 USB controllers to the rpi, connect a USB WiFi adapter if you have one or an ethernet cable if you don't
- Lastly, connect the power supply
- Hopefully you will now see a your rpi booting up on your monitor. Exciting!!

After a couple minutes of many commands running in the terminal, you might see a graphical welcome screen telling you if any gamepads are detected.  This is the EmulationStation graphical user interface, and this is where you'll spend most of your future time - it is the entry point to all the games. It will also be used to configure the controllers. But we'll do that later, first let's do some other rpi setup.  

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
- Press `Ctrl`+`x` to exit (if you made any changes, you will also need to press `Y` and then `Enter` to save the changes)
- Run `sudo nano /etc/wpa_supplicant/wpa_supplicant.conf`
- The file should have these two lines (if not, add them)
```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
```  
- Add the following lines at the end of the file (replace `<NETWORK NAME>` and `<NETWORK PASSWORD>` with your network's name and password). Note: you might find that some keys you press on the keyboard do not type the correct key on the screen. For example, for me when I press on `"` it actually types `@`, so when I want to type `"` I press on `@` on the keyboard instead. A little annoying, but we'll fix that in a couple minutes.
```
network={
ssid="<NETWORK NAME>"
psk="<NETWORK PASSWORD>"
}
```
- Press `Ctrl`+`x`, `Y`, `Enter` to save the file
- Run `sudo ifdown wlan0` followed by `sudo ifup wlan0`
- You might see error messages a-la "Operation not permitted" or "Invalid argument". Ignore those. I hate ignoring error messages, but it's actually fine.
- Run `ping google.com` to see if you have internet.  If you get a response, great. If not, reboot rpi with `sudo reboot` (press F4 when rpi boots back up to get back to the terminal) and now you should have internet

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
To make WPA2-Enterprise network work, I also added `auto wlan0` to the `/etc/network/interfaces` file, but I'm not sure if that's required or not.

### Install vim editor
This is a personal choice, you don't have to do this, but I prefer vim over nano as my text editor so I want to use it from now on when editing files... `sudo apt-get install -y vim`

### Set up raspberry pi
We'll use a raspberry pi configuration script to do some basic setup first.
- Run `sudo raspi-config`
- Select `1 Expand Filesystem`, let it run for a second
- Select `Internationalisation Options`
- Inside Internationalisation Options, select `Change Locale` (it takes a few seconds to open up)
- Assuming you're not in the UK, scroll down to the `en_GB` entry that is selected and press `space` to unselect it. Now scroll to `en_CA.UTF-8` (select UTF, not ISO!) and select it. Press `Enter`.
- If you get prompted with a "warning" about changing the default language, **do** select `en_CA` instead of `None`. This will take a minute.
- Go back to `Internationalisation Options`, now select `Change Timezone` and choose yours (for me: `America`, `Vancouver`)
- Go back to `Internationalisation Options`, now select `Change Keyboard Layout`, then `Generic 105-key (Intl) PC`, then `Other`, then `English (US)`, then `English (US)` again. Now choose the default option for the next 3 dialogs and wait a minute while the settings change.
- Optional: go to `Overclock` and select a higher setting to increase performance
- Go to `Advanced Options` then `Update` - this will update the configuration script (the one that we're currently using) to the latest version (this could take several minutes if you're on a slow WiFi)
- When it's done, go to `Finish` and reboot

### Set up RetroPie
Now we'll do a basic set up of RetroPie
- Run `sudo RetroPie-Setup/retropie_setup.sh`
- Select `UPDATE RetroPie Setup script`
- **For this step, you need to have one and only one USB controller plugged in to your rpi**. I'm not sure if it matters if you have more than one controller plugges in, but I was having problems at some points and it was fine when I had only one controller plugged in. Go to `SETUP` and select `Register RetroArch controller`. You need to press on the button within a second, or else it'll assume you don't have that button. For any button that it asks that you don't have on your controller, simply wait a couple seconds until it times out.  If you pressed the wrong button and missed one, just run the tool again after it finishes.

### Small configuration to bash profile
You can skip this step, I just wanted to add a few aliases for `ll` and for quickly restarting/shutting off raspberry pi.  Run `sudo vim ~/.bashrc` and add the following to the end of the file
```
alias ll='ls -l'  # use `ll` to list the files/folders in the current directory in a long ling
alias reboot='sudo shutdown -r now'  # use `reboot` to restart your raspberry pi
alias shutdown='sudo shutdown -h now'  # use `shutdown` to turn off your raspberry pi safely
```
Type `:wq` to save and close the file

### Configure special controller hotkeys
Not mandatory, but I find it very useful.  Edit `/opt/retropie/emulators/retroarch/retroarch.cfg` (using `sudo vim` or any other editor) and add the following lines to the end of the file
```
input_enable_hotkey_btn = "6"
input_exit_emulator_btn = "7"
input_save_state_btn = "2"
```
This defines two htokeys: exit a game, and save/load state.  
Wn you're inside a game, there is no way to exit using the keyboard (at least none that I'm aware of), which means that you need to unplug your rpi to shut it off if you want to exit from a game. Here we defined a hotkey that will exit a game if we press buttons "6" and "7" together, which for me are the "select" and "start" buttons on the SNES controller.  Button 6 ("select") is the hotkey in this case, and button 7 is the exit button.  The other hotkey we defined is select+X (X = "2" for me, again - you can change that to anything else, or even remove that line if you don't want this).  This key combination will bring up a screen during play that allows us to save/load.


### Test run EmulationStation
Okay that was enough setup, let's play!  RetroPie comes with a few games builtin, so let's just get a taste of what we have.  We can get to EmulationStation (the graphical user interface that lets you browse through your consoles/games) either by typing `emulationstation` in the terminal or by rebooting.  Let's reboot, just to make sure all our settings are loaded. Make sure to plug in all the controllers you have. After reboot you'll get to the welcome screen, but this time don't press F4.

EmulationStation will tell you to press any key to configure the controller. Do that.  When you finish, you'll be brought to a screen with all the consoles/emulators you have available. Move left/right to PORTS, and select DOOM 1 (or whatever game). Using the first controller, press "select"+"start" to ensure that our configurations worked and hopefully it exited the game and came back to ES.  If not, try it on the other controller. If it doesn't work.... then Google :)


### Adding games
Press F4 to go back to the terminal.  Now run `ls RetroPie/roms`. You'll see a list of all available emulators.  If you want to play Super Mario World on SNES, then you need to download the ROM for Super Mario World and place it inside the `snes` folder. If you have a GameBoy Colour game, the ROM goes inside the `gbc` folder.  Then when you go back to ES, you'll see your new game. But how do you get the ROM files into these folders?  There are several options:  

- You can download the ROM directly from within rpi (either using `wget` or using a browser), I haven't tried it but it should work. Not my recommendation.
- You can use Google how to transfer files using samba, but I haven't done this. Not my recommendation.
- Use FTP. This will work assuming both your rpi and your laptop are connected to the same network. Follow these steps:
  - Download the game ROMs that you want
  - Download an FTP client such as FileZilla
  - Enable SSH on your rpi: type `sudo raspi-config`, go to `Advanced Options`, select `SSH` and select Enable 
  - Find out what's the IP of your rpi using `hostname -I`
  - Open FileZilla. In the "Host" input, type the IP address. Username is "pi" and password is "raspberry". Port is 22. Click connect.
  - Transfer your ROMs from your laptop to the appropriate folders on the rpi (each ROM should go under `/home/pi/RetroPie/roms/<emulator>`)
  - On your rpi, type `emulatorstation` and enjoy!
- **My recommendation: Nice little trick - if you plug a USB stick into your rpi and wait a minute, rpi will automatically add all the `roms` folders onto your USB. Now you can put the USB into your laptop, place ROMs inside their correct folders on the USB stick, and when you put the USB back into your rpi, rpi will automagically copy the ROMs from the USB onto it.  This seems a little magical to me so I personally prefer the next method.**

Note: if you want to see a list of all emulators that ES supports, you can look inside `/etc/emulationstation/es_systems.cfg`.  You can also look there to see what file type each emulator expects for its ROMs.
 
### Improving performance to be able to run newer games such as PS1

I found that changing a few settings gave me very good performance with some games that otherwise were not playable.  First, I set overclocking to Turbo (`sudo raspi-config` --> Overclock --> Turbo).  Then I added a few settings to the configuration (`sudo vim /boot/config.txt`):

```
overscan_scale=1
arm_freq=1000
core_freq=500
sdram_freq=600
over_voltage=6
avoid_safe_mode=1
gpu_mem=120
force_turbo=0
```

## Getting SNES Bomberman multiplayer working

It seems like the multiplayer feature on Super Bomberman for SNES does not work when the ROM is in the `snes` folder where all my other SNES ROMs are.  When I move Bomberman to use the pisnes emulator by moving the ROM to the `snes-pisnes` folder, then multiplayer does work, but some buttons on the controller don't work. I have to use the keyboard to start or end games, which is annoying. Turns out that the pisnes controller configuration isn't inherting its controls properly, and I need to manually set the key mappings for pisnes.  

- Edit the pisnes config file with `sudo vim /opt/retropie/emulators/pisnes/snes9x.cfg`
- Scroll down to the `[Joystick]` section
- Comment out all the buttons from `A` to `SELECT`
- Add the following button settings:

```
A_1=0
B_1=1
X_1=2
Y_1=3
L_1=4
R_1=5
SELECT_1=6
START_1=7
```

Now when you go back to ES, you will see two different entries for Super Nintendo which is not ideal, but at least all games will work properly :)

There may be other games that also experience similar problems, maybe they will also work better if placed under a different emulator. I know it at least works for Super Bomberman.

### Misc quick info
- What's my rpi IP address? `hostname -I`
- Rpi settings `sudo raspi-config`
- Go to the GUI `startx`
- Start EmulationStation: `emulationstation` (or `exit` from the terminal will also work)
- Default user name/pw: pi/raspberry
- Exit from ES to terminal: F4
- Switch terminal windows: `ctrl`+`alt`+(`F1`-`F6`)
- Exit from game to ES with controller: `select`+`start`
- Save/load inside game: `select`+`X`
- What emulators are supported/what file extensions they expect for ROMs `/etc/emulationstation/es_systems.cfg`
- RetroPie settings: `sudo ~/RetroPie-Setup/retropie_setup.sh`
- Controller settings: `/opt/retropie/emulators/retroarch/retroarch.cfg` and `/opt/retropie/configs/all/retroarch.cfg` - I'm not quite sure yet what the difference is
- To test what each key on your controller maps to: `jstest /dev/input/js0`

## Known bugs with current setup
- Mario Kart on N64 doesn't work
- The desktop/GUI doesn't seem to work to well
