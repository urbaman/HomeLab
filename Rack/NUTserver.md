# NUT UPS Server

plug in ups

```bash
lsusb
```

should see something like

```bash
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 001 Device 019: ID 09ae:2012 Tripp Lite
Bus 001 Device 002: ID 2109:3431 VIA Labs, Inc. Hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```


```bash
sudo apt update
sudo apt install nut nut-client nut-server
sudo nut-scanner -U
```

should see something like

tripp lite

```bash
[nutdev1]
        driver = "usbhid-ups"
        port = "auto"
        vendorid = "09AE"
        productid = "2012"
        product = "Tripp Lite UPS"
        vendor = "Tripp Lite"
        bus = "001"
```

apc 1500

```bash
[nutdev1]
        driver = "usbhid-ups"
        port = "auto"
        vendorid = "051D"
        productid = "0002"
        product = "Back-UPS XS 1500M FW:947.d10 .D USB FW:d10"
        serial = "3xxxxxxxxxxx"
        vendor = "Tripp Lite"
        bus = "001"
```

apc 850

```bash
[nutdev3]
        driver = "usbhid-ups"
        port = "auto"
        vendorid = "051D"
        productid = "0002"
        product = "Back-UPS ES 850G2 FW:931.a10.D USB FW:a"
        serial = "3xxxxxxxxxxx"
        vendor = "American Power Conversion"
        bus = "001"
```

```bash
sudo nano /etc/nut/ups.conf
```

```bash
pollinterval = 1
maxretry = 3

[tripplite]
    driver = usbhid-ups
    port = auto
    desc = "Tripp Lite 1500VA SmartUPS"
    vendorid = 09ae
    productid = 2012

[apc-network]
    driver = usbhid-ups
    port = auto
    desc = "APC Back-UPS XS 1500"
    vendorid = 051d
    productid = 0002
    serial = 3xxxxxxxxx

[apc-modem]
    driver = usbhid-ups
    port = auto
    desc = "APC 850 VA"
    vendorid = 051d
    productid = 0002
    serial = 3xxxxxxxxx
```


```bash
sudo nano /etc/nut/upsmon.conf
```

```bash
MONITOR tripplite@localhost 1 admin secret master
MONITOR apc-modem@localhost 1 admin secret master
MONITOR apc-network@localhost 1 admin secret master
```

```bash
sudo nano /etc/nut/upsd.conf
```

```bash
local host

LISTEN 127.0.0.1 3493 
all interface

LISTEN 0.0.0.0 3493 
```

```bash
sudo nano /etc/nut/nut.conf
```

```bash
MODE=netserver
sudo nano /etc/nut/upsd.users
```

```bash
[monuser]
  password = secret
  admin master


```bash
sudo nano /etc/udev/rules.d/99-nut-ups.rules
```

```bash
SUBSYSTEM!="usb", GOTO="nut-usbups_rules_end"

# TrippLite
#  e.g. TrippLite SMART1500LCD - usbhid-ups
ACTION=="add|change", SUBSYSTEM=="usb|usb_device", SUBSYSTEMS=="usb|usb_device", ATTR{idVendor}=="09ae", ATTR{idProduct}=="2012", MODE="664", GROUP="nut", RUN+="/sbin/upsdrvctl stop; /sbin/upsdrvctl start"

LABEL="nut-usbups_rules_end"
```

```bash
sudo service nut-server restart
sudo service nut-client restart
sudo systemctl restart nut-monitor
sudo upsdrvctl stop
sudo upsdrvctl start
```

APC UPS 950 va

query device by USB bus

```bash
lsusb -D /dev/bus/usb/001/057
```

```bash
Device Descriptor:
  bLength                18
  bDescriptorType         1
  bcdUSB               2.00
  bDeviceClass            0
  bDeviceSubClass         0
  bDeviceProtocol         0
  bMaxPacketSize0        64
  idVendor           0x051d American Power Conversion
  idProduct          0x0002 Uninterruptible Power Supply
  bcdDevice            0.90
  iManufacturer           1
  iProduct                2
  iSerial                 3
  bNumConfigurations      1
  Configuration Descriptor:
    bLength                 9
    bDescriptorType         2
    wTotalLength       0x0022
    bNumInterfaces          1
    bConfigurationValue     1
    iConfiguration          0
    bmAttributes         0xe0
      Self Powered
      Remote Wakeup
    MaxPower                2mA
    Interface Descriptor:
      bLength                 9
      bDescriptorType         4
      bInterfaceNumber        0
      bAlternateSetting       0
      bNumEndpoints           1
      bInterfaceClass         3 Human Interface Device
      bInterfaceSubClass      0
      bInterfaceProtocol      0
      iInterface              0
        HID Device Descriptor:
          bLength                 9
          bDescriptorType        33
          bcdHID               1.00
          bCountryCode           33 US
          bNumDescriptors         1
          bDescriptorType        34 Report
          wDescriptorLength    1049
         Report Descriptors:
           ** UNAVAILABLE **
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x81  EP 1 IN
        bmAttributes            3
          Transfer Type            Interrupt
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0008  1x 8 bytes
        bInterval             100
```


## NUT CGI Server

```bash
sudo apt install apache2 nut-cgi
sudo nano /etc/nut/hosts.conf
```

```bash
MONITOR tripplite@localhost "Tripp Lite 1500VA SmartUPS - Rack"
MONITOR apc-modem@localhost "APC 850 VA - Wall"
MONITOR apc-network@localhost "APC Back-UPS XS 1500 - Rack"
```

```bash
sudo a2enmod cgi
sudo systemctl restart apache2
sudo nano /etc/nut/upsset.conf
```

```bash
I_HAVE_SECURED_MY_CGI_DIRECTORY
```

visit

```bash
http://your.ip.adddress/cgi-bin/nut/upsstats.cgi
```
