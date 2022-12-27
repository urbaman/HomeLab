# IPMI tools

apt install ipmitool freeipmi-tools freeipmi-bmc-watchdog freeipmi-ipmidetect lm-sensors fancontrol read-edid i2c-tools python-smbus
 
Query Dell's Third-Party PCIe card based default system fan response:

ipmitool raw 0x30 0xce 0x01 0x16 0x05 0x00 0x00 0x00

response like below means Disabled

16 05 00 00 00 05 00 01 00 00

response like below means Enabled

16 05 00 00 00 05 00 00 00 00

Jets off or Set Third-Party PCIe Card Default Cooling Response Logic To Disabled:

ipmitool raw 0x30 0xce 0x00 0x16 0x05 0x00 0x00 0x00 0x05 0x00 0x01 0x00 0x00 

Jets on or Set Third-Party PCIe Card Default Cooling Response Logic To Enabled:

ipmitool raw 0x30 0xce 0x00 0x16 0x05 0x00 0x00 0x00 0x05 0x00 0x00 0x00 0x00
