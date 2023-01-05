# IPMI tools

To add ipmitools to Proxmox (debian 11), let's install it and see some commands:

```bash
apt install ipmitool freeipmi-tools freeipmi-bmc-watchdog freeipmi-ipmidetect lm-sensors fancontrol read-edid i2c-tools
 ```

Query Dell's Third-Party PCIe card based default system fan response:

```bash
ipmitool raw 0x30 0xce 0x01 0x16 0x05 0x00 0x00 0x00
```

response like below means Disabled

```bash
16 05 00 00 00 05 00 01 00 00
```

response like below means Enabled

```bash
16 05 00 00 00 05 00 00 00 00
```

Jets off or Set Third-Party PCIe Card Default Cooling Response Logic To Disabled:

```bash
ipmitool raw 0x30 0xce 0x00 0x16 0x05 0x00 0x00 0x00 0x05 0x00 0x01 0x00 0x00 
```

Jets on or Set Third-Party PCIe Card Default Cooling Response Logic To Enabled:

```bash
ipmitool raw 0x30 0xce 0x00 0x16 0x05 0x00 0x00 0x00 0x05 0x00 0x00 0x00 0x00
```

## iDRAC Setting Fan Speed on Poweredge R730xd with IPMI (should work with others but commands will vary with device names

_Note when you change fan settings, give it a few seconds to happen, it isn't quite instant
get list of devices.  Also "lanplus" is a specification, not your local interface name, so leave it as lanplus._

```bash
ipmitool -I lanplus -H 192.168.16.2 -U sysadmin -P ******** sdr list
```

### get fan speeds, exhaust temp  (use dev names from above cmd)

```bash
ipmitool -I lanplus -H 192.168.16.2 -U sysadmin -P ******** sensor reading "Fan1 RPM" "Fan2 RPM" "Fan3 RPM" "Fan4 RPM" "Fan5 RPM" "Fan6 RPM" "Exhaust Temp"
```

### change fan speed control to manual

```bash
ipmitool -I lanplus -H 192.168.16.2 -U sysadmin -P ******** raw 0x30 0x30 0x01 0x00
```

### set fan speed to 30% of max (hint: 0x1e is hexadecimal for 30)

```bash
ipmitool -I lanplus -H 192.168.16.2 -U sysadmin -P ******** raw 0x30 0x30 0x02 0xff 0x1e
```

### set fan speed to 40% of max (hint: 0x28 is hexadecimal for 40)

```bash
ipmitool -I lanplus -H 192.168.16.2 -U sysadmin -P ******** raw 0x30 0x30 0x02 0xff 0x28
```

### change fan speed control to automatic (if you want to go back to letting it manage itself and be unbearably loud again)

```bash
ipmitool -I lanplus -H 192.168.16.2 -U sysadmin -P ******** raw 0x30 0x30 0x01 0x01
```

Check CPU1 temp

```bash
ipmitool -I lanplus -H 192.168.16.2 -U sysadmin -P ******** sdr elist | grep "Temp" | grep "3.1" | cut -c 39-40
```

Check CPU2 temp

```bash
ipmitool -I lanplus -H 192.168.16.2 -U sysadmin -P ******** sdr elist | grep "Temp" | grep "3.2" | cut -c 39-40
```

## Script for fan speed management

This script disables the Third-Party PCIe Card Default Cooling Response Logic, then checks CPU temps and sets the fan speed to a fitting value.

Create the log file and write "100" in it

```bash
nano /etc/fanspeedcontrol/fanspeed.old
```

```bash
100
```

Create a /usr/sbin/fanmanager file

```bash
nano /usr/sbin/fanmanager
```

```bash
#!/bin/bash

# Define variables
oldspeedfile=/etc/fanspeedcontrol/fanspeed.old
changed=0

# Deactivate Third-Party PCIe Card Default Cooling Response
ipmitool raw 0x30 0xce 0x00 0x16 0x05 0x00 0x00 0x00 0x05 0x00 0x01 0x00 0x00 >/dev/null

# Set the fan speed control to manual
ipmitool -I lanplus -H 192.168.1.223 -U root -P Zo3m8zNRGN9$ raw 0x30 0x30 0x01 0x00 >/dev/null

# Check the CPU temperatures
cpu1temp=$(ipmitool -I lanplus -H 192.168.1.223 -U root -P Zo3m8zNRGN9$ sdr elist | grep "Temp" | grep "3.1" | cut -c 39-40)
cpu2temp=$(ipmitool -I lanplus -H 192.168.1.223 -U root -P Zo3m8zNRGN9$ sdr elist | grep "Temp" | grep "3.2" | cut -c 39-40)
cputemp=$(bc <<< "scale=2; ($cpu1temp + $cpu2temp) / 2")

# Set the fan speed to proper value
if (( $(bc <<< "$cputemp <= 49") > 0 ));
then
  if grep -Fxq "10" $oldspeedfile
  then
    echo "Not changed" >/dev/null
  else
    newspeed=10
    oldspeed=$(cat $oldspeedfile)
    sed -i "s/$oldspeed/$newspeed/g" $oldspeedfile
    changed=1
    ipmitool -I lanplus -H 192.168.1.223 -U root -P Zo3m8zNRGN9$ raw 0x30 0x30 0x02 0xff 0xB >/dev/null
    message="CPU1 Temp is $cpu1temp, CPU2 Temp is $cpu2temp; fan speed set to 11%"
  fi
elif (( $(bc <<< "$cputemp > 49") > 0 && $(bc <<< "$cputemp <= 51") > 0 ));
then
  if grep -Fxq "20" $oldspeedfile
  then
    echo "Not changed" >/dev/null
  else
    newspeed=20
    oldspeed=$(cat $oldspeedfile)
    sed -i "s/$oldspeed/$newspeed/g" $oldspeedfile
    changed=1
    ipmitool -I lanplus -H 192.168.1.223 -U root -P Zo3m8zNRGN9$ raw 0x30 0x30 0x02 0xff 0x14 >/dev/null
    message="CPU1 Temp is $cpu1temp, CPU2 Temp is $cpu2temp; fan speed set to 20%"
  fi
elif (( $(bc <<< "$cputemp > 51") > 0 && $(bc <<< "$cputemp <= 53") > 0 ));
then
  if grep -Fxq "30" $oldspeedfile
  then
    echo "Not changed" >/dev/null
  else
    newspeed=30
    oldspeed=$(cat $oldspeedfile)
    sed -i "s/$oldspeed/$newspeed/g" $oldspeedfile
    changed=1
    ipmitool -I lanplus -H 192.168.1.223 -U root -P Zo3m8zNRGN9$ raw 0x30 0x30 0x02 0xff 0x1E >/dev/null
    message="CPU1 Temp is $cpu1temp, CPU2 Temp is $cpu2temp; fan speed set to 30%"
  fi
elif (( $(bc <<< "$cputemp > 53") > 0 && $(bc <<< "$cputemp <= 55") > 0 ));
then
  if grep -Fxq "40" $oldspeedfile
  then
    echo "Not changed" >/dev/null
  else
    newspeed=40
    oldspeed=$(cat $oldspeedfile)
    sed -i "s/$oldspeed/$newspeed/g" $oldspeedfile
    changed=1
    ipmitool -I lanplus -H 192.168.1.223 -U root -P Zo3m8zNRGN9$ raw 0x30 0x30 0x02 0xff 0x28 >/dev/null
    message="CPU1 Temp is $cpu1temp, CPU2 Temp is $cpu2temp; fan speed set to 40%"
  fi
elif (( $(bc <<< "$cputemp > 55") > 0 && $(bc <<< "$cputemp <= 57") > 0 ));
then
  if grep -Fxq "50" $oldspeedfile
  then
    echo "Not changed" >/dev/null
  else
    newspeed=50
    oldspeed=$(cat $oldspeedfile)
    sed -i "s/$oldspeed/$newspeed/g" $oldspeedfile
    changed=1
    ipmitool -I lanplus -H 192.168.1.223 -U root -P Zo3m8zNRGN9$ raw 0x30 0x30 0x02 0xff 0x32 >/dev/null
    message="CPU1 Temp is $cpu1temp, CPU2 Temp is $cpu2temp; fan speed set to 50%"
  fi
elif (( $(bc <<< "$cputemp > 57") > 0 && $(bc <<< "$cputemp <= 59") > 0 ));
then
  if grep -Fxq "60" $oldspeedfile
  then
    echo "Not changed" >/dev/null
  else
    newspeed=60
    oldspeed=$(cat $oldspeedfile)
    sed -i "s/$oldspeed/$newspeed/g" $oldspeedfile
    changed=1
    ipmitool -I lanplus -H 192.168.1.223 -U root -P Zo3m8zNRGN9$ raw 0x30 0x30 0x02 0xff 0x3C >/dev/null
    message="CPU1 Temp is $cpu1temp, CPU2 Temp is $cpu2temp; fan speed set to 60%"
  fi
elif (( $(bc <<< "$cputemp > 59") > 0 && $(bc <<< "$cputemp <= 61") > 0 ));
then
  if grep -Fxq "70" $oldspeedfile
  then
    echo "Not changed" >/dev/null
  else
    newspeed=70
    oldspeed=$(cat $oldspeedfile)
    sed -i "s/$oldspeed/$newspeed/g" $oldspeedfile
    changed=1
    ipmitool -I lanplus -H 192.168.1.223 -U root -P Zo3m8zNRGN9$ raw 0x30 0x30 0x02 0xff 0x46 >/dev/null
    message="CPU1 Temp is $cpu1temp, CPU2 Temp is $cpu2temp; fan speed set to 70%"
  fi
elif (( $(bc <<< "$cputemp > 61") > 0 && $(bc <<< "$cputemp <= 63") > 0 ));
then
  if grep -Fxq "80" $oldspeedfile
  then
    echo "Not changed" >/dev/null
  else
    newspeed=80
    oldspeed=$(cat $oldspeedfile)
    sed -i "s/$oldspeed/$newspeed/g" $oldspeedfile
    changed=1
    ipmitool -I lanplus -H 192.168.1.223 -U root -P Zo3m8zNRGN9$ raw 0x30 0x30 0x02 0xff 0x50 >/dev/null
    message="CPU1 Temp is $cpu1temp, CPU2 Temp is $cpu2temp; fan speed set to 80%"
  fi
elif (( $(bc <<< "$cputemp > 63") > 0 && $(bc <<< "$cputemp <= 65") > 0 ));
then
  if grep -Fxq "100" $oldspeedfile
  then
    echo "Not changed" >/dev/null
  else
    newspeed=100
    oldspeed=$(cat $oldspeedfile)
    sed -i "s/$oldspeed/$newspeed/g" $oldspeedfile
    changed=1
  #ipmitool -I lanplus -H 192.168.1.223 -U root -P Zo3m8zNRGN9$ raw 0x30 0x30 0x02 0xff 0x5A >/dev/null
  #message="CPU1 Temp is $cpu1temp, CPU2 Temp is $cpu2temp; fan speed set to 90%"
    ipmitool -I lanplus -H 192.168.1.223 -U root -P Zo3m8zNRGN9$ raw 0x30 0x30 0x01 0x01 >/dev/null
    message="CPU1 Temp is $cpu1temp, CPU2 Temp is $cpu2temp; fan speed set to dynamic"
  fi
#elif (( $(bc <<< "$cputemp > 65") > 0 && $(bc <<< "$cputemp <= 67") > 0 ));
#then
#  ipmitool -I lanplus -H 192.168.1.223 -U root -P Zo3m8zNRGN9$ raw 0x30 0x30 0x02 0xff 0x64
#  message="CPU1 Temp is $cpu1temp, CPU2 Temp is $cpu2temp; fan speed set to 100%"
fi

if (( $changed == 1 ))
then
  echo $message | mail -s "pvenode1: fan control" pveadmin@urbaman.it
else
  echo $message >/dev/null
fi
```

Set the file executable

```bash
chmod a+x /usr/sbin/fanmanager
```

Set it to run every 5 minutes in crontab

```bash
crontab -e
```

```bash
*/5 * * * * /usr/sbin/fanmanager
```
