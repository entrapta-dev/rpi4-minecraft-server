# Minecraft Server on Raspberry Pi 4

## Minimum Requirements
- Raspberry Pi 4 2GB
- USB Type-C Power Adapter
- 16GB Micro SD Card

More RAM on the Pi and a bigger SD Card is nice but not required.  
If you plan on running mods or have a lot of players on the server, do go for higher.

---

## Set up and update Raspberry Pi 4

### Prepare SD Card

Download the latest Raspberry Pi OS Lite from the official site.  
We're going for the 64-bit lite version as it takes up less space and we're running this headless anyway.  
https://www.raspberrypi.com/software/operating-systems/#raspberry-pi-os-64-bit  

1. Flash the image on your SD Card.  
2. Mount the `boot` partition of the SD Card.  
3. Create an empty file called `ssh`. You can do this in a terminal with `touch ssh`.
4. Unmount the SD Card and place it in the Raspberry Pi.

### Set Up and Update Raspberry Pi OS

Boot up the Pi and log in.  

Ensure everything is up to date:  
```
sudo apt update && sudo apt upgrade
```

Reboot.

---

# Set Up the Minecraft Server

## Preparation

Install the latest Java Runtime:  
```
sudo apt install openjdk-17-jdk
```

Reboot.

---

## Installing

We're using the open source version called [PaperMC](https://github.com/PaperMC/Paper) instead of the official Minecraft server files because it uses less resources and is a little easier on our Raspberry Pi.  

**Note:** Mods and modpacks __**do not work**__ on this version. If you really want mods, install the official server files instead:  
Copy the link address from the [official website](https://launcher.mojang.com/v1/objects/c8f83c5655308435b3dcf03c06d9fe8740a77469/server.jar) and paste that in the `wget` command.

All the instructions are the same apart from the `wget` command and the `jar` file will be differently named.  
Be sure to put it in a separate folder if you are installing both versions for some reason.  

Create a folder to put the server files into and change into that folder.  
e.g.: `mkdir papermc-server`, `cd papermc-server`.  

Download the latest version by going to the [papermc download page](https://papermc.io/downloads) and right click on the latest version number and selecting `Copy Link Address` or similar.  

In your terminal use the following command to download the server files into your newly created folder:  
```
wget -O papermc.jar https://papermc.io/api/v2/projects/paper/versions/1.18.2/builds/250/downloads/paper-1.18.2-250.jar
```
Do *not* just copy paste the above command. Be sure to use the link to the latest version you copied before.  

After it is done downloading we run the server once like so:  
```
java -Xms1024M -Xmx1024M -jar papermc.jar nogui
```
It will stop pretty quickly as we'll have to accept the EULA first.  
It will have now created a `eula.txt`. Open it and edit the single uncommented line so it'll read `eula=true`.

Start the server again:  
```
java -Xms1024M -Xmx1024M -jar papermc.jar nogui
```
**Note:** The `Xms` and `Xmx` sets the minimum and maximum reserved amount of RAM for the server respectively.  
It is recommended to keep both values identical.
It is also recommended to keep both values at half the amount of RAM in your device.  
e.g.: If you have a 4GB Raspberry Pi 4, use `-Xms2048M -Xmx2048M` instead.

World generation can take a while. Wait for it to finish and it should say `Done!` in the log along with how much time it took.  

This is all that is necessary for a basic server with all default settings to run.  
All of the following is technically *optional* but I advice to read through it anyway and make your own customizations as you see fit for your server.  

---

## Configuration

Be sure the server is not running before making any customizations.  
If needed, shut down the server with the `stop` command.  

### Server Settings

Edit `server.properties` to adjust your server settings like PVP and difficulty. For example, take note of the settings for:  
```
level-name=world
motd=A Minecraft Server
pvp=true
difficulty=easy
max-players=10
white-list=false
enforce-whitelist=false
```
View all the commands and what they do here:  https://minecraft.fandom.com/wiki/Server.properties#Java_Edition_3  

I also recommend looking at game rules where you can set things such as enabling/disabling drowning damage, day/night cycle, and much more.  
View all possible game rules here:  https://minecraft.fandom.com/wiki/Commands/gamerule  

**Note:** If you want your server to be publicly available over the internet it is necessary to set up port forwarding on your router. As this process differs per router I advice you to look around your router's settings or check the manual if you don't know how.  
You will need to forward port number `25565` on both TCP and UDP for internal and external and set the IP address of your Minecraft server where required.  
Be sure to update your `server.properties` with this information. e.g.:   
```
server-port=25565
server-ip=192.168.1.123
```

### Assign Server Operators

Edit `ops.json` and add any user you wish to be a server operator (OP).  
Example config:  
```
[
  {
    "uuid": "fa251abf-e71a-96cd-18e4-3d1abc4f0c2a",
    "name": "PlayerName",
    "level": 4,
  }
]
```
You can get someone's `uuid` from the log when they log onto the server.

---

# Overclocking the Raspberry Pi 4

## Preparation

Update the Raspberry Pi OS:  
```
sudo apt update && sudo apt upgrade
```

## Check Raspberry Pi Model and Revision

Run:  
```
cat /proc/cpuinfo
```

Default output:  
```
processor       : 0
BogoMIPS        : 108.00
Features        : fp asimd evtstrm crc32 cpuid
CPU implementer : 0x41
CPU architecture: 8
CPU variant     : 0x0
CPU part        : 0xd08
CPU revision    : 3

processor       : 1
BogoMIPS        : 108.00
Features        : fp asimd evtstrm crc32 cpuid
CPU implementer : 0x41
CPU architecture: 8
CPU variant     : 0x0
CPU part        : 0xd08
CPU revision    : 3

processor       : 2
BogoMIPS        : 108.00
Features        : fp asimd evtstrm crc32 cpuid
CPU implementer : 0x41
CPU architecture: 8
CPU variant     : 0x0
CPU part        : 0xd08
CPU revision    : 3

processor       : 3
BogoMIPS        : 108.00
Features        : fp asimd evtstrm crc32 cpuid
CPU implementer : 0x41
CPU architecture: 8
CPU variant     : 0x0
CPU part        : 0xd08
CPU revision    : 3

Hardware        : BCM2835
Revision        : b03114
Serial          : 10000000647c7dac
Model           : Raspberry Pi 4 Model B Rev 1.4
```

To just quickly check the model run:  
```
cat /proc/device-tree/model
```

---

## Voltage and Frequency Check

Create a `stats.sh` script:
```
echo -e "\e[1;33mStatistics:\e[0m" && echo -e Current Frequency - "$id:\t$(cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_cur_freq)" && echo -e Minimum Frequency - "$id:\t$(cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_min_freq)" && echo -e Maximum Frequency - "$id:\t$(cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_max_freq)" && for id in core sdram_c sdram_i sdram_p ; do echo -e Voltage - "$id:\t$(vcgencmd measure_volts $id)" ; done && echo -e ARM Frequency - "$id:\t$(vcgencmd measure_clock arm)" && echo -e Core Frequency - "$id:\t$(vcgencmd measure_clock arm)" && echo -e Temperature - "$id:\t$(vcgencmd measure_temp)"
```

### Default output:

```
Current Frequency - sdram_p:    600000
Minimum Frequency - sdram_p:    600000
Maximum Frequency - sdram_p:    1800000
Voltage - core: volt=0.8900V
Voltage - sdram_c:      volt=1.1000V
Voltage - sdram_i:      volt=1.1000V
Voltage - sdram_p:      volt=1.1000V
ARM Frequency - sdram_p:        frequency(48)=600117184
Core Frequency - sdram_p:       frequency(48)=600169920
Temperature - sdram_p:  temp=34.5'C
```

### Overclocked output:  

```
NYA
```

---

## Monitoring

### Temperature Monitoring

Create a `temps.sh` script to monitor temperature every two seconds:  
```
watch -n 2 vcgencmd measure_temp
```

Uncooled default should run about 54'C

---

### Frequency Monitoring

Create a `freqs.sh` script to monitor frequency every second:  
```
watch -n 1 vcgencmd measure_clock arm
```

---

### Monitor Everything

Create a `monitor.sh` scripts to monitor all important stats:  
```
watch -d -n 2 'vcgencmd measure_clock arm ; vcgencmd measure_temp'
```

Default output should look similar to:  
```
frequency(48)=600169920
temp=34.5'C
```

Overclocked output should look similar to:  
```
NYA
```

Check and make note of the default frequency and voltage settings by running the `stats.sh` script we created earlier.  

---

## Overclocking

Edit the main configuration file:  
```
sudo nano /boot/config.txt
```

Find:  
```
#uncomment to overclock the arm. 700 MHz is the default.
#arm_freq=800
```

Change it to:  
```
#uncomment to overclock the arm. 700 MHz is the default.
over_voltage=6
arm_freq=2000
```

Reboot.

It should now run at 2GHz speed.  

Verify this by running `stats.sh` or `monitor.sh` again.  

**Note:** It may not be immediately apparent as it will need load to increase the frequency clock. Start the Minecraft server you just set up and you should see it go up.
