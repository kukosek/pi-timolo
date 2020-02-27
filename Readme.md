# fork by me
I customized pi-timolo by Claude Pageau to suit my needs. I am making a long time timelapse (~1 year) of a tree in front of my house. I wanted to take a photo every 10 minutes that will be uploaded to google drive just after capturing it. I found pi-timolo, which didn't suit my needs perfectly, but it's written in python, so I can modify it pretty easily.
## flashybrid
I know SD card corruption is a thing on rpi, so I decided to write to the SD as little as possible. I set up flashybrid. This is a tool that makes your root filesystem a ramdisk, so all the log files and things wouldn't have to be written directly to the SD card. On shutdown, it syncs directories you selected to the SD card. You can also sync the files to SD with the command `sudo fh-sync`

## changes
* it works only with python3 cause i am using new functions like subprocess.run(). so make sure to set it up to work with python3. Follow the guide on the pi-timolo wiki.
* `takeDayImage()` and `takeNightImage()` in _pi-timolo.py_
  + The idea of pi-timolo remote directories is about adding an rclone script to cron, so it would run like every 10 mins. I didn't really like that, so I changed the code to run my rclone script after every timelapse capture.
  + Script it would run is configurable in _config.py_, entry runScriptAfterCapture
* new function `getTlNumFromRclone()` in _pi-timolo.py_
  + I (happily quickly) realized that when the pi would have an power loss, the number of timelapse image wouldn't be synced to the SD card (remember i have flashybrid?). The bad thing that happens next is that it would start sending files with filenames that arleady are in the gdrive folder - rclone will of course replace them.
  + I wrote a function that is ran at the start of main. It gets a list of files in the rclone directory (i know, it would be more efficent if it was just the newest file, but rclone doesn't have an option for that as far as I know) and extracts the timelapseNum from the newest (just alphabetically) filename. It also skips files that don't look like timelapse images. In case of an rclone error it just retries forever.
  + Configuration of this feature in _config.py_:
    - _timelapseNumGetFromRcloneRemote_ set to True/False if wanna to enable the whole functionality
    - _timelapseListRcloneCmd_ the rclone command to get the list of files in your directory
    - _timelapseListRcloneOutputSplit_ rclone ls outputs a line for every file. on the line are some information. the informations are splitted by a whitespace (" "). if it outputs just the filename, you can type False and skip the next option
    - _timelapseListRcloneOutputIndex_ the filename is the last information on the line for me (-1)
    - _timelapseListRcloneErrorResetNetworking_ whether (True/False) to reset networking in case of an rclone error. you must type your sudo password down here. that may create a security hole :{
    - _timelapseListRcloneErrorRetry_ whether to retry the listing in case of an error
    - _timelapseListRcloneErrorRetrySleep_ time in seconds to sleep before retrying
    - _raspiSudoPassword_ your sudo password, in case _timelapseListRcloneErrorResetNetworking_ is True
* new rclone script _rclone-tl-copy-remove.sh_ that copies the directory to remote then deletes all the local files except  the newest one for previewin. At next run the files that are not local should stay on the remote because it uses `rclone sync` instead of `rclone copy`
* changed _pi-timolo.sh_ to run _pi-timoly.py_ with python3 instead of python2.7.
* I don't know why but you can't start the program from the _menubox.sh_ script. so you must start it with the command `./pi-timolo.sh start` in the folder where it is located 
# PI-TIMOLO [![Mentioned in Awesome <INSERT LIST NAME>](https://awesome.re/mentioned-badge.svg)](https://github.com/thibmaek/awesome-raspberry-pi)
### Raspberry (Pi)camera, (Ti)melapse, (Mo)tion, (Lo)wlight 
## For Details See [Program Features](https://github.com/pageauc/pi-timolo/wiki/Introduction#program-features) and [Wiki Instructions](https://github.com/pageauc/pi-timolo/wiki) and [YouTube Videos](https://www.youtube.com/playlist?list=PLLXJw_uJtQLa11A4qjVpn2D2T0pgfaSG0)

***IMPORTANT:*** Raspbian Stretch and pi-timolo.py ver 11.11 and earlier has long exposure low light 
camera freezing issue due to kernel panic that requires a reboot to gain
control of camera back per https://github.com/waveform80/picamera/issues/528 
pi-timolo.py ver 11.12 has a fix to resolve issue but
requires the latest Raspbian firmware. If you encounter camera freeze with latest Stretch image then
you will need to run ***sudo rpi-update*** to update Stretch to latest firmware.  Normal
backup precautions are advised before doing the firmware update.  See [wiki](https://github.com/pageauc/pi-timolo/wiki/Basic-Trouble-Shooting#raspbian-stretch-kernel-panic-and-camera-freeze)
***Note:*** Raspbian Jessie works fine and does Not encounter freezing issue with long exposure low light operation.

* ***Release 9.x*** New Features have been Added. See Wiki Details below    
 [plugins Setup and Operation](https://github.com/pageauc/pi-timolo/wiki/How-to-Use-Plugins)   
 [Rclone Setup and Media Sync](https://github.com/pageauc/pi-timolo/wiki/How-to-Setup-rclone) (Replaces gdrive)    
 [watch-app.sh Remote Configuration Management](https://github.com/pageauc/pi-timolo/wiki/How-to-Setup-config.py-Remote-Configuration)   
 [python3 Support Details](https://github.com/pageauc/pi-timolo/wiki/Prerequisites#python-3-support)   
* ***Release 10.x*** Added Sched Start to Motion Track, Timelapse and VideoRepeat. See Wiki Details below    
 [How To Schedule Motion, Timelapse or VideoRepeat](https://github.com/pageauc/pi-timolo/wiki/How-to-Schedule-Motion,-Timelapse-or-VideoRepeat)  
 This release requires config.py be updated by the user with config.py.new since new variables have been added.
* ***Release 11.12*** Added adhoc fix for Debian Stretch kernel panic and camera freeze issue when running
 under very low light conditions.  Note a ***sudo rpi-update*** may be required to update firmware if freezing
 still occurs under pi-timolo.py ver 11.12 or greater
 
## Requirements
Requires a [***Raspberry Pi computer***](https://www.raspberrypi.org/documentation/setup/) and a 
[***RPI camera module installed***](https://www.raspberrypi.org/documentation/usage/camera/).
Make sure hardware is tested and works. Most [RPI models](https://www.raspberrypi.org/products/) will work OK. 
A quad core RPI will greatly improve performance due to threading. A recent version of 
[Raspbian operating system](https://www.raspberrypi.org/downloads/raspbian/) is Recommended.
 
## Quick Install or Upgrade
**IMPORTANT** - It is suggested you do a Raspbian ***sudo apt-get update*** and ***sudo apt-get upgrade***
before curl install, since it is **No longer** performed by the pi-timolo-install.sh script

***Step 1*** With mouse left button highlight curl command in code box below. Right click mouse in **highlighted** area and Copy.     
***Step 2*** On RPI putty SSH or terminal session right click, select paste then Enter to download and run script.     

    curl -L https://raw.github.com/pageauc/pi-timolo/master/source/pi-timolo-install.sh | bash

The command above will download and Run the GitHub ***pi-timolo-install.sh*** script. 
An upgrade will not overwrite configuration files.   

* ***NOTICE*** gdrive is no longer installed with pi-timolo-install.sh, I have been testing
rclone and it is Now the Default. Some ***rclone-*** samples are included. Make a copy of one, rename and edit for
your own needs.  See [Wiki - How to Setup Rclone](https://github.com/pageauc/pi-timolo/wiki/How-to-Setup-rclone).
Note: If a ***/usr/local/bin/gdrive*** File Exists, It Will Remain. Older files are still available on this GitHub Repo.   

## Test Install
To Test Run default config.py - motion track(HD image) plus timelapse(5 min interval). 
 
    cd ~/pi-timolo
    ./pi-timolo.py

### For More Details see [Basic Trouble Shooting](https://github.com/pageauc/pi-timolo/wiki/Basic-Trouble-Shooting) or [pi-timolo Wiki](https://github.com/pageauc/pi-timolo/wiki)

## Description
PI-TIMOLO is primarily designed for ***headless operation*** and includes rclone that
can securely synchronize specified media folders and files with a users remote storage service of choice. This works well for remote security and monitoring
cameras. Camera config.py and conf settings can be easily administered remotely from a designated sync directory using ***watch-app.sh***
script using a crontab entry to periodically check for updates between the pi-timolo camera and a users remote storage rclone service name. 

pi-timolo is python 2/3 compatible and can take timelapse and/or motion detection images/videos, separately or together. Will take
long exposure Night (lowlight) images for Time Lapse and/or Motion. Has relatively smooth twilight transitions based on a threshold light
setting, so a real time clock is not required. Customization settings are saved in a ***config.py*** and conf files and optional special
purpose plugin config files. Optional plugin feature allows overlaying config.py settings with custom settings for specific tasks.  

Includes ***makevideo.sh*** to create timelapse or motion lapse videos from images, ***convid.sh*** to convert/combine 
h264 to mp4 format, a simple minumum or no setup web server to view images or videos and ***menubox.sh*** 
to admin settings and stop start pi-timolo and webserver as background tasks. 
       
For more Details see [Github Wiki](https://github.com/pageauc/pi-timolo/wiki)   

## Minimal Upgrade
If you are just interested in a minimal upgrade (must have pi-timolo previously installed)
from a logged in ssh or terminal session execute the following commands.  

    cd ~/pi-timolo
    sudo apt-get install python-opencv
    cp config.py config.py.old
    cp pi-timolo.py pi-timolo.py.old
    wget -O config.py https://raw.github.com/pageauc/pi-timolo/master/source/config.py
    wget -O pi-timolo.py https://raw.github.com/pageauc/pi-timolo/master/source/pi-timolo.py    
    
Edit config.py to transfer any customized settings from config.py.old  
    
## Manual Install or Upgrade  
From logged in RPI SSH session or console terminal perform the following. You can review
the pi-timolo-install.sh script code before executing.

    cd ~
    wget https://raw.github.com/pageauc/pi-timolo/master/source/pi-timolo-install.sh
    more pi-timolo-install.sh    # Review code if required
    chmod +x pi-timolo-install.sh
    ./pi-timolo-install.sh
    
## Menubox
pi-timolo has a whiptail administration menu system. The menu's allow
start/stop of pi-timolo.py and/or webserver.py as background tasks, as well as
editing configuration files, making timelapse videos from jpg images, converting or joining mp4 files Etc.    

To run menubox.sh from ssh console or terminal session execute commands below.

    cd ~/pi-timolo
    ./menubox.sh

![menubox main menu](menubox.png)
 
## Webserver
I have also written a standalone LAN based webserver.py to allow easy access to pi-timolo image and video files
on the Raspberry from another LAN computer web browser.  There is no setup required but the display
settings can be customized via variables in the config.py file or via menubox admin menuing.     
***NOTE:*** webserver.py is normally run in background using menubox.sh, webserver.sh or from /etc/rc.local     
To Test Run from ssh console or terminal session. 
    
    cd ~/pi-timolo
    ./webserver.py

![webserver browser screen shot](webserver.jpg)
 
## Reference Links  
Detailed pi-timolo Wiki https://github.com/pageauc/pi-timolo/wiki  
YouTube Videos https://www.youtube.com/playlist?list=PLLXJw_uJtQLa11A4qjVpn2D2T0pgfaSG0
 
Good Luck
Claude Pageau 
