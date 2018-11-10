# psx-pi-smbshare
psx-pi-smbshare began with the intent of allowing SMB sharing to Multiman and Open Playstation Loader from a Raspberry Pi.  It has evolved into a Pi-based swiss army knife for enhancing classic game consoles.  

## How it works
psx-pi-smbshare is a preconfigured Raspbian based image for Raspberry Pi 1, 2, and  3.  It runs a [Samba](https://en.wikipedia.org/wiki/Samba_(software)) share, a pi-compatible build of [ps3netsrv](https://github.com/dirkvdb/ps3netsrv--), and reconfigures the ethernet port to act as a router.  This gives low-latency, direct access to the Samba service through an ethernet cable connection between a PS2/PS3 and Raspberry Pi.  This configuration is achieved by running [setup.sh](/setup.sh).  A pre-supplied [image](https://github.com/toolboc/psx-pi-smbshare/releases/) can be applied directly to a Micro-SD card using something like [etcher.io](https://etcher.io/).  The image allows you to use the full available space on the SD card after the OS is first booted.

An [Xlink Kai](http://www.teamxlink.co.uk/) client is also included and accessible on the device at http://smbshare:34522/.  This allows for multi-player gaming over extended LAN.  The service is possible to use on a variety of devices including PS2, PS3, Xbox, Xbox 360, and Gamecube.  Just connect an ethernet cable to your game console and access the Xlink Kai Service over Wi-Fi with a smart phone, tablet, or computer.

## What you can do with it
psx-pi-smbshare works out of the box on PS3 with [MultiMAN](http://www.psx-place.com/threads/update2-multiman-v-04-81-00-01-02-base-update-stealth-for-cex-dex-updates-by-deank.12145/).  This functionality allows you to stream and backup up various games and media to the Samba share service running on the Raspberry Pi.

psx-pi-smbshare also works out of the box on PS2 with [Open Playstation Loader](https://github.com/ifcaro/Open-PS2-Loader) and supports streaming of PS2 backups located on the Samba share service. It can also work with [POPStarter for SMB](https://bitbucket.org/ShaolinAssassin/popstarter-documentation-stuff/wiki/smb-mode) to allow streaming of PS1 games from Open Playstation Loader.

psx-pi-smbshare supports an ability to route traffic from the ethernet port through a wireless network connection to the outside world.  With this configuration, the XLink Kai Service can be used on pretty much any device with an ethernet port.  This includes Xbox, Xbox 360, PS2, PS3, and Gamecube.  There is also support for Ad-Hoc multiplayer on PSP using XLink Kai.  

# Quickstart

*Prerequisites*
* Raspberry Pi 1, 2, or 3
* Micro-SD Card (8GB+ suggested)

## Flash the image
Download the latest [psx-pi-smbshare release image](https://github.com/toolboc/psx-pi-smbshare/releases/) and burn it to a Micro-SD card with [etcher.io](http://etcher.io)

## Configuring Wireless Network
If you wish to configure the wireless network on a Raspberry Pi 2 or 3, you need to add a file to **/boot** on the Micro-SD card.  

Create a file on **/boot** named **wpa_supplicant.conf** and supply the following (change country to a [valid 2 letter code](https://en.wikipedia.org/wiki/ISO_3166-1)):

    ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
    update_config=1
    country=US
    
    network={
            ssid="<SSID>"
            psk="<PASSWORD>"
    }

When the pi is next booted, it will attempt to connect to the wireless network in this configuration.  You are then able to access the raspberry pi on the network and allow for outbound connections from a PS2/PS3 over the wireless network.  
The raspberry pi is configured to have a hostname `smbshare` with a user `pi` and a password of `raspberry`.  

## Accessing the XLink Kai Service
Visit http://smbshare:34522/ or http://<YOUR_PSX_PI_SMBSHARE_DEVICE_IP>:34522/

## Accessing the SMB Share
With a wireless network configured, you should be able to access the SMB share by visiting `\\SMBSHARE\share` on windows or `smb://smbshare/share` on Mac / Linux.

![Accessing SMB](/Assets/smbshare.PNG)

The share is preconfigured with a folder structure to accomodate ps3netsrv and Open Playstation Loader expected file paths.

## Accessing USB drive(s) on the SMB Share
Plug and play auto-sharing of USB storage devices over SMB is supported:

* USB Drives are automounted to the /media/<Partition> directory

* When a USB drive is plugged in, the USB Drive becomes available on the SMB Share @ `\\SMBSHARE\share` 
    
* When a USB drive is removed, the device falls back to sharing the Micro-SD card @ `\\SMBSHARE\share` 

## Forwarding Active FTP session to a connected device
Assuming your console / device has an ip of 192.168.2.2, you may run the following script to forward an Active FTP session:

        sudo iptables -t nat -A PREROUTING -p tcp --dport 21 -j DNAT --to-destination 192.168.2.2:21
        sudo modprobe ip_nat_ftp ports=21

## Configuring for use with MultiMAN on PS3

*Prerequisites*
* Playstation 3 running a [recent release of MultiMAN](http://store.brewology.com/ahomebrew.php?brewid=24)

*Steps*
1. Connect the pi ethernet port into the ethernet port of the PS3 and power the pi using the PS3 usb or an external power supply 
2. In the PS3 XMB select "Settings" => "Network Settings" => "Internet Connection Settings" and configure to connect using the ethernet connection as follows:
    
    "Internet Connection Settings" => "Custom" => "Wired" => "Auto-Detect" => "Manual"

            IP Address = 192.168.2.2
            Subnet Mask = 255.255.255.0
            Default Router = 192.168.2.1
            Primary DNS = 8.8.8.8
            Secondary DNS = <leave blank or use your home router ip address>

    "Automatic" => "Do Not Use" => "Enable"
3. Launch MultiMAN
4. Select "Settings" => "Network Servers"
5. Configure using the Ip Address '192.168.2.1' (ip to the smbshare assigned by dhcp server running on the Pi) and Port '38008' (default)
6. You should see new section for the network server under 'Photos' / 'Music' / 'Video' / 'Retro' and a new location to copy games to when using copy ISO in the 'Games' menu.  

PS3 Games backed up to the network server can be found and loaded from the "Games" menu in MultiMAN.
PS1, PS2, and PSP games can be found and loaded from "Retro" => "PSONE" "PS2" OR "PSP"  
PS2 backups must be loaded from the HDD but can be copied directly to the SMB server.

## Configuring for use with Open Playstation Loader

*Prerequisites*
* Playstation 2 fat or slim running a [recent release of Open Playstation Loader](http://www.ps2-home.com/forum/viewtopic.php?p=29251#p29251) 

*Steps*
1. Connect the pi ethernet port into the ethernet port of the PS2 and power the pi using the PS2 usb or an external power supply 
2. Boot Open Playstation Loader and select "Settings" => "Network Config".  
Ensure that the following options are set:

        Ethernet Link Mode = Auto
        PS2 
            IP address type = Static
            IP address = 192.168.2.2
            Mask = 255.255.255.0
            Gateway = 192.168.2.1
            DNS Server = 8.8.8.8
        SMB Server
            Address Type = IP
            Address = 192.168.2.1
            Port = 445
            Share = share
            Password = <not set>

    ![PS2 OPL Settings](/Assets/PS2-OPL-settings.png)

Don't forget to select "Save Config" when you return to "Settings"

3. Reconnect or restart Open Playstation Loader
4. PS2 Games will be listed under "ETH Games".  To add PS2 games, copy valid .iso backups to `\\SMBSHARE\share\DVD` or `\\SMBSHARE\share\CD`

## Configuring for use with POPSLoader on Open Playstation Loader

*Prerequisites*
* Ensure that you have successfully followed the steps above for "Configuring for use with Open Playstation Loader"

*Steps*
1. Download the [ps2 network modules](https://bitbucket.org/ShaolinAssassin/popstarter-documentation-stuff/downloads/network_modules.7z) 
2. Extract the POPSTARTER folder 
3. Modify IPCONFIG.DAT to:
        
        192.168.2.2 255.255.255.0 192.168.2.1
4. Modify SMBCONFIG.DATA to:
        
        192.168.2.1 share
5. Copy the POPSTARTER folder to your memory card
6. Hop on the internet and look for a copy of a file named "POPS_IOX.PAK" with md5sum "a625d0b3036823cdbf04a3c0e1648901" and copy it to `\\SMBSHARE\share\POPS`.  This file is not included for "reasons".
7. PS1 backups must be converted to .VCD and run through a special renaming program in order to show up in OPL.

    To convert .bin + .cue backups, you can use the included "CUE2POP_2_3.EXE" located in `\\SMBSHARE\share\POPS\CUE2POPS v2.3`
    Copy your .VCD backups to `\\SMBSHARE\share\POPS` then run `\\SMBSHARE\share\POPS\OPLM\OPL_Manager.exe` to rename your files appropriately.
    
    Once converted and properly renamed, your games will show up under the "PS1 Games" section of OPL

    A detailed guide is available @ http://www.ps2-home.com/forum/viewtopic.php?f=64&t=5002

## Playing Halo 2 online with Xlink Kai on Xbox

*Prerequisites*
* An original Xbox or Xbox 360 with backwards compatibility support
* A copy of Halo 2
* An Xlink Kai account from http://www.teamxlink.co.uk/

*Steps*
1. Burn the [latest psx-pi-smbshare image](https://github.com/toolboc/psx-pi-smbshare/releases) to a Micro-SD card
2. Configure Wi-fi per the steps above in ["Configuring the Wireless Network"](https://github.com/toolboc/psx-pi-smbshare#configuring-wireless-network)
3. Plug the pi into the Xbox ethernet port and verify that you are able to obtain an ip automatically in Network Settings
4. Vist the Xlink Kai service running on the pi @ http://smbshare:34522 or http://<YOUR_PSX_PI_SMBSHARE_DEVICE_IP>:34522/ and login with your Xlink Kai account
5. Select an available Halo game from the Xlink Kai portal (there are usually a few running in South America)
6. Launch Halo 2 and select "System Link"
7. Join a game and have fun!

## Playing SOCOM 2 online with Xlink Kai on PS2

*Prerequisites*
* A Fat PS2 with Network Adapter or Slim PS2
* A copy of SOCOM 2 for PS2
* An Xlink Kai account from http://www.teamxlink.co.uk/

*Steps*
1. Burn the [latest psx-pi-smbshare image](https://github.com/toolboc/psx-pi-smbshare/releases) to a Micro-SD card
2. Configure Wi-fi per the steps above in ["Configuring the Wireless Network"](https://github.com/toolboc/psx-pi-smbshare#configuring-wireless-network)
3. Plug the pi into the PS2 ethernet port
4. The following setup needs to be performed one time:  Boot your PS2 up with SOCOM 2 and select "Online" at the title screen. Once you hit the first blue screen hit edit network configuration to be sent to the network setup. Now delete any old network settings and create a new one. Using Automatic settings is fine but you may wish to set the following manual settings if you have issues:

        Console IP: 192.168.2.2
        Subnet Mask: 255.255.255.0
        Router IP/Gateway: 192.168.2.1
5. Vist the Xlink Kai service running on the pi @ http://smbshare:34522 or http://<YOUR_PSX_PI_SMBSHARE_DEVICE_IP>:34522/ and login with your Xlink Kai account
6. Select an available SOCOM 2 game from the Xlink Kai portal (there is usually one running at all times)
7. Head back to the SOCOM 2 title screen and select "LAN"
8. Join a game and have fun!

## Using a Second WiFi interface as an Access Point to Xlink Kai 
*Prerequisites*
* 1 external wifi dongle for RPi 2/3 or 2 external wifi dongles for RPi 1

*Steps*
1. Burn the [latest psx-pi-smbshare image](https://github.com/toolboc/psx-pi-smbshare/releases) to a Micro-SD card
2. Plug in the external wifi dongle(s)
3. Configure Wi-fi per the steps above in ["Configuring the Wireless Network"](https://github.com/toolboc/psx-pi-smbshare#configuring-wireless-network)
4. Configure the device to connect to "XlinkKai" SSID when the pi has booted using Password `XlinkKai` 

Note: XlinkKai will only work on one network interface (wifi or ethernet) at a time and will lock onto the first interface connected to from a compatible device until reboot

## Playing PSP games online with Xlink Kai on PSP

*Prerequisites*
* A wifi capable PSP
* 1 external wifi dongle for RPi 2/3 or 2 external wifi dongles for RPi 1 (*tested with an [Edimax-EW-7811Un](https://www.amazon.com/Edimax-EW-7811Un-150Mbps-Raspberry-Supports/dp/B003MTTJOY) wifi dongle*)
* A Multiplayer game which supports Ad-Hoc 
* An Xlink Kai account from http://www.teamxlink.co.uk/

*Steps*
1. Burn the [latest psx-pi-smbshare image](https://github.com/toolboc/psx-pi-smbshare/releases) to a Micro-SD card
2. Configure Wi-fi per the steps above in ["Configuring the Wireless Network"](https://github.com/toolboc/psx-pi-smbshare#configuring-wireless-network)
3. SSH to you psx-pi-smbshare instance using the default username `pi` and default password `raspberry`
4. Ensure that your PSP is set to Automatic in Network Settings under Ad Hoc Mode
5. Run the following commands to disable the hostapd access point and enable Ad-Hoc Wifi:
        
        sudo service hostapd stop
        sudo iw wlan1 set type ibss

7. Start an Ad-Hoc multiplayer session from a game on the PSP
6. Run the follwoing command 
    
        sudo iw wlan1 scan | grep PSP_ -B 5

    You will receive an output similar to:

        freq: 2462
        beacon interval: 100 TUs
        capability: IBSS ShortPreamble (0x0022)
        signal: -42.00 dBm
        last seen: 0 ms ago
        SSID: PSP_S000000001_L_GameShar

    Take note of the frequency and SSID
7. Using the information in the previous step, execute the following while the multiplayer session is waiting:

        sudo iw wlan1 ibss join <SSID> <frequency> 

    Ex: sudo iw wlan1 ibss join PSP_S000000001_L_GameShar 2462

    If you receive "Operation not supported (-95)" then your wifi adapter may not be compatible

    Once you know this command, you can re-use the following script at any time to enable PSP Ad-Hoc mode:
        
        sudo service hostapd stop
        sudo iw wlan1 set type ibss
        sudo iw wlan1 ibss join <SSID> <frequency>

    Note: You must run this script after starting an Ad-Hoc multiplayer session on the PSP

8. Exit the Ad-Hoc multiplayer session and start a new one
9. Vist the Xlink Kai service running @ http://smbshare:34522 or http://<YOUR_PSX_PI_SMBSHARE_DEVICE_IP>:34522/ and login with your Xlink Kai account
10. Select "Metrics" and scroll down to "Found Consoles" and you should see your PSP device
11. Select an available PSP game from the Xlink Kai portal 
12. Join a game and have fun!

# Demos
[Launching PS2 Backups with OPL](https://www.youtube.com/watch?v=FJEdWW6YhJo&feature=youtu.be)

# Credits
Thx to the following:
* Jay-Jay for [OPL Daily Builds](https://github.com/Jay-Jay-OPL/OPL-Daily-Builds) 
* danielb for [OPLM](http://www.ps2-home.com/forum/viewtopic.php?f=64&t=189)
* dirkvdb for [ps3netsrv--](https://github.com/dirkvdb/ps3netsrv--)
* arpitjindal97 for [wifi-to-eth-route.sh](https://github.com/arpitjindal97/raspbian-recipes/blob/master/wifi-to-eth-route.sh)
* Team Xlink for [Xlink Kai](http://www.teamxlink.co.uk/)
* Pul Gasari for [Testing streaming games from Raspberry Pi to PS2 using psx-pi-smbshare](https://www.youtube.com/watch?v=FJEdWW6YhJo&feature=youtu.be)
