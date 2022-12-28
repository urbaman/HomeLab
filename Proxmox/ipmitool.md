# IPMI tools

To add ipmitools to Proxmox (debian 11), let's install it and see some commands:

```bash
apt install ipmitool freeipmi-tools freeipmi-bmc-watchdog freeipmi-ipmidetect lm-sensors fancontrol read-edid i2c-tools python-smbus
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
