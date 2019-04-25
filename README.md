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

### 2. Octoprint

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
# sudo mount -t cifs //192.168.1.156/LPC1768 /mnt/lan -o username=Ruedi,password=020Sa123  ## IP/networkshare, username and password of the local machine
# sudo cp /mnt/lan/firmware.bin /mnt/firmware.bin
# sudo umount -t cifs //192.168.1.156/LPC1768 /mnt/lan

sudo umount -t vfat /dev/sda1 /mnt  ##UNmount Printerboards sdcard
echo "M997" >> /dev/ttyACM0  ##reboot the Printerboard and initialize the firmware
```

```/home/pi/bin/FWupdate```

```Shell
#!/bin/bash
sudo /usr/local/bin/updateFW
exit 0
```

