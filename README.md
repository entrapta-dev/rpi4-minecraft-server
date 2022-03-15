# Minecraft Server on Raspberry Pi 4

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
Temperature - sdram_p:  temp=53.5'C
```

### Overclocked output:  

```
NYA
```


---

## Temperature Monitoring

Create a `temps.sh` script to monitor temperature every two seconds:  

```
watch -n 2 vcgencmd measure_temp
```

Uncooled default should run about 54'C

---
