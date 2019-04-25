# 32bit_FWupdate_for_Octoprint
Bashscript for Firmware Update on 32Bit Boards (Marlin)

At first, it is not Plug & Play and it is more "Quick and Dirty" than "Nice and Clean". But it is my way I managed a "One click Firmware Update" for my SKR V1.3 Board, and it works. Yay!

## Baseline

- I have my Marlin Firmware Files on Google Drive, so i can access it from different pc's.
- My 3D Printer is connected to my WiFi over a RPi 3B+ with Octoprint running on it.
- I use PlatformIO to compile the 32bit Marlin Firmware

## My Goal was

- making firmwarechanges
- hit PIO Upload
- press one Button in Octoprint
- Done!

## Setup

Here I descibe my Setup, i don't know if it works with other Setups.
It is possible to work without Google Drive and send the firmware.bin from the local Machine, but more on this later in the Setup. 

### 1. PlatformIO

#### Platformio.ini
In the enviroment Section of your Processor, in my case [env:LPC1768], it is possible to define additional Upload Ports.
By default, the compiled File is stored in the Folder ```...\.pioenvs\LPC1768\firmware.bin```if you hit Compile. If you click PIO Upload, PlatformIO stores the File in the folder and tries to Upload to the Board, if it is connected via USB. If no Board is present, the file will be uploaded to the additionaly defined Upload Ports. In My case, I defined one Port for each of my PC's.

<p align="center"><img  alt="platformio.ini" src="Images/PIO_ini.PNG"></p>

### 2. Octopi

#### Scripts

I don't know exactly why, but i needed two Scripts. The first Script ```updateFW``` contains all the stuff like mount SDcard, backup the Firmware, download the ```firmware.bin```and push it to the SDcard, unmount and reset the Board. The second one ```FWupdate``` only calls the first Scrip and give the ```exit 0```Status back to Octoprint.

Here are the Memory Locations and the Scripts

```/usr/local/bin/updateFW```

```Shell
#!/bin/bash
sudo mount -t vfat /dev/sda1 /mnt  ##mount Printerboards SDcard
cp --backup=numbered /mnt/* /home/pi/Documents/Firmware_Backup  ##Backup all files on the SDcard
sudo rm -f /mnt/*  ##clear all files on the SDcard

##download the firmware-file from GoogleDrive online
fileid="FileID_from_share_link"  ##copy and paste the part after "id=" of the "share-link" something like "0B3nIRrOFv6hVcE9IWEI1QnVZYjT" keep quotation marks
filename="/mnt/firmware.bin"  ##destination path and filename  keep quotation marks
curl -c ./cookie -s -L "https://drive.google.com/uc?export=download&id=${fileid}" > /dev/null
curl -Lb ./cookie "https://drive.google.com/uc?export=download&confirm=`awk '/download/ {print $NF}' ./cookie`&id=${fileid}" -o ${filename}

##alternativly download the firmware-file from a local workstation
# sudo mount -t cifs //192.168.XXX.XXX/LPC1768 /mnt/lan -o username=NAME,password=PASSWORD  ## IP/networkshare, username and password of the local machine
# sudo cp /mnt/lan/firmware.bin /mnt/firmware.bin
# sudo umount -t cifs //192.168.XXX.XXX/LPC1768 /mnt/lan

sudo umount -t vfat /dev/sda1 /mnt  ##UNmount Printerboards sdcard
echo "M997" >> /dev/ttyACM0  ##reboot the Printerboard and initialize the firmware
```


```/home/pi/bin/FWupdate```

```Shell
#!/bin/bash
sudo /usr/local/bin/updateFW
exit 0
```

Make sure the Rights are set correctly and the Scripts are executable

```$ sudo chmod 755 /path/to/file```

### Sudoers

Normally every Command executetd with ```sudo``` needs a Password. Since we can not enter the Password while the Script is running, we need to tell the Systen to execute the ```sudo```Commands within these two Scripts without a Password entry. Therefore we create a file in the Directorie ```/etc/sudoers.d/```. The Name of this File is non-essential. We name it ```updatescript```. Type the folowing in the Terminal:

```Shell
$ sudo visudo -f /etc/sudoers.d/updatescript
```
write these two lines into the File
```Shell
pi ALL=(ALL:ALL) NOPASSWD: /usr/local/bin/updateFW
pi ALL=(ALL:ALL) NOPASSWD: /home/pi/bin/FWupdate
```
Safe the changes with ```CTRL```+```O``` and exit the editor with ```CTRL```+```X```

Make sure the Rights for this file are set to 440 and the owner is root
```$ sudo chmod 440 /path/to/file```
```$ sudo chown root /path/to/file```

### 3. Octoprint

The last step is to configure the "System Command Editor". After the installation over Pluginmanager, open the Settings and rightclick on the green Frame. Choose ```Create Command``` and configure it like this:

<p align="center"><img  alt="Sys Cmd Editor" src="System_Command_Editor.JPG"></p>

Hit Safe and You're Done!
Here is Your "Updade Button"
<p align="center"><img  alt="Update Button" src="update_button.png"></p>
