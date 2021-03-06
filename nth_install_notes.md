# nth install notes  

# Start with RasPi OS buster desktop image

`2021-05-07-raspios-buster-armhf.img` is what I used


### make terminal ASCII art  

`sudo nano /etc/motd`

```
           _|      _|
_|_|_|   _|_|_|_|  _|_|_|
_|    _|   _|      _|    _|
_|    _|   _|      _|    _|
_|    _|     _|_|  _|    _|   by denki-oto

```

### RASPI-CONFIG

`sudo raspi-config`  

set display options -> underscan to yes

set locale and wifi country


### COMPILE DTS OVERLAY

```
wget https://raw.githubusercontent.com/okyeron/nth/main/nth-buttons-encoders-overlay.dts
sudo dtc -W no-unit_address_vs_reg -@ -I dts -O dtb -o /boot/overlays/nth-buttons-encoders-overlay.dtbo nth-buttons-encoders-overlay.dts
# ignore warning
```

### Modify boot/config.txt for inputs, etc.

`sudo nano /boot/config.txt`  

See [Pi_Audio_Setup.md](Pi_Audio_Setup.md) for setting up audio  


```
# Buttons and encoders device tree
dtoverlay=nth-buttons-encoders-overlay

# safe shutdown
dtoverlay=gpio-shutdown,gpio_pin=26

# MIDI UART
enable_uart=1
dtoverlay=pi3-miniuart-bt
dtoverlay=midi-uart0

```

### ROTATE THE DISPLAY

```
# on Raspberry Pi OS 5.4 +
# Rotate display in Desktop > Preferences > Screen Configuration

# Then edit 40-libinput.conf for touch screen rotation
# Reference: https://github.com/swkim01/waveshare-dtoverlays

sudo nano /usr/share/X11/xorg.conf.d/40-libinput.conf
```

### add to touchscreen entry

`	Option "TransformationMatrix" "-1 0 1 0 -1 1 0 0 1"`

touchscreen inputclass should look like this:  
```
Section "InputClass"
        Identifier "libinput touchscreen catchall"
        MatchIsTouchscreen "on"
        MatchDevicePath "/dev/input/event*"
        Driver "libinput"
        Option "TransformationMatrix" "-1 0 1 0 -1 1 0 0 1"
EndSection
```

### display details 
	https://www.waveshare.com/wiki/5inch_DSI_LCD    
	https://www.raspberrypi.org/documentation/accessories/display.html#tips-and-tricks  


### INSTALL PACKAGES

```
sudo apt-get update
sudo apt-get -y install vim git bc i2c-tools input-utils libncurses5 alsa-utils libi2c-dev
sudo apt-get -y install python3-pip

sudo apt install --no-install-recommends jackd2
sudo apt-get install libboost1.67-dev libjack-jackd2-dev, jack-tools
sudo apt install -y libnanomsg-dev supercollider-language supercollider-server supercollider-supernova supercollider-dev 
sudo apt install -y sc3-plugins ladspalist 
sudo apt install -y liblua5.3-dev libudev-dev libevdev-dev liblo-dev libcairo2-dev liblua5.3-dev libavahi-compat-libdnssd-dev libasound2-dev 
sudo apt install usbmount

# patchage
sudo apt-get install -y patchage

# OpenFrameworks
sudo apt-get install -y xorg xorg-dev libx11-dev libluajit-5.1-dev swig mesa-utils libgl1-mesa-glx libgles2-mesa libgles2-mesa-dev libegl1-mesa-dev libgbm-dev dphys-swapfile libffi-dev

# SDL2
sudo apt-get install -y libsdl2-dev libsdl2-image-dev libsdl2-ttf-dev

# joyosc
sudo apt-get install -y libtinyxml2-dev

# monome stuff (optional?)
sudo apt install -y libmonome-dev serialosc

## my fork of ttymidi
git clone https://github.com/okyeron/ttymidi
 
	# Compare with changes on https://github.com/oxytu/ttymidi and https://github.com/owenosborn/ttymidi
	
	## make it
	
	cd ~/ttymidi
	make
	sudo make install

	## run with 

	ttymidi -s /dev/ttyAMA0 &

	# install ttymidi service
	wget https://raw.githubusercontent.com/okyeron/nth/main/ttymidi.service
	sudo cp -f ttymidi.service /etc/systemd/system/ttymidi.service
	rm ttymidi.service


## amidiminder https://github.com/mzero/amidiminder

git clone https://github.com/mzero/amidiminder.git
cd amidiminder
make
sudo dpkg -i build/amidiminder.deb

```

### remove modemmanager ?
`sudo apt-get remove -y modemmanager`



### MODEP  
```
curl https://blokas.io/apt-setup.sh | sh
sudo apt-get update
sudo apt-get install modep

sudo systemctl disable pisound-ctl.service
sudo systemctl disable pisound-btn.service
```

change jack.service - or edit `/etc/jackdrc` - change `hw:pisound` to `hw:0`  

`sudo apt-get install modep-common modep-mod-ui modep-mod-host`




### Bluetooth ?
```
sudo systemctl start bluetooth
sudo systemctl status bluetooth
rfkill list
# if blocked
sudo rfkill unblock bluetooth
sudo systemctl restart bluetooth

bluetoothctl
power on
scan on
devices
pair XX:XX:XX:XX
trust XX:XX:XX:XX
connect XX:XX:XX:XX
```

### EYESY

```
git clone https://github.com/okyeron/EYESY_OS_for_RasPi.git Eyesy
cd Eyesy
./deploy.sh
```

### M8 headless
```
sudo apt update && sudo apt install -y git gcc make libsdl2-dev libserialport-dev

hw:CARD=sndrpiproto,DEV=0
hw:CARD=M8,DEV=0

/usr/bin/jackd -R -P 95 -d alsa -d hw:0 -r 48000 -n 3 -p 512 -S -s &


https://github.com/laamaa/m8c

jackd -d alsa -d hw:M8 -r44100 -p512 &
alsa_out -j m8out -d hw:0 -r 44100 &
jack_connect system:capture_1 m8out:playback_1
jack_connect system:capture_2 m8out:playback_2


jackd -d alsa -d hw:M8 -r48000 -p512 &
alsa_out -j m8out -d hw:0 -r48000 &
jack_connect system:capture_1 m8out:playback_1
jack_connect system:capture_2 m8out:playback_2
```
