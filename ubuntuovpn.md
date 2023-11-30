# How to Install & Connect OpenVPN Client on Ubuntu

Are you looking to install and connect OpenVPN Client on Ubuntu? It’s easy to do! In this guide, we’ll walk you through the process step-by-step. Assuming one of your clients wants to secure a connection to their server. The client has configured OpenVPN server on their network and provide you client configuration file. By the end, you’ll have a secure, encrypted connection to your VPN server. So, let’s get started!

## Prerequisites

Before you can install OpenVPN Client on Ubuntu, you’ll need to make sure you have the following:

- The latest version of Ubuntu is installed on your computer.
- A reliable internet connection.
- An OpenVPN configuration file, which you can get from your VPN provider.
- A working VPN account.
- Once you have all the prerequisites in place, you’re ready to start installing OpenVPN Client on Ubuntu.

## Step 1 – Installing OpenVPN Client on Ubuntu

Installing OpenVPN Client on Ubuntu is relatively easy. Just follow the steps below and you should be up and running in no time.

Open a terminal window on your Ubuntu machine and type in the following command: `sudo apt install openvpn`. This will install the OpenVPN package on your system.

```bash
sudo apt update && sudo apt install apt-transport-https curl -y
sudo curl -fsSL https://packages.openvpn.net/packages-repo.gpg | sudo tee /etc/apt/keyrings/openvpn.asc
sudo DISTRO=$(lsb_release -c | awk '{print $2}')
sudo echo "deb [signed-by=/etc/apt/keyrings/openvpn.asc] https://packages.openvpn.net/openvpn3/debian $DISTRO main" | sudo tee /etc/apt/sources.list.d/openvpn-packages.list
sudo apt update && sudo apt install openvpn3 -y
```

Once the package is installed, you’ll need to create a configuration file. To do this, type in the following command in the terminal window:

```bash
sudo nano $HOME/.openvpn3/client.conf 
```

This will open a text editor where you can paste the configuration file (opvn) from your VPN provider.

Once you’ve pasted the configuration file, save and exit the text editor by pressing Ctrl+X followed by Y and then Enter. Then, import the configuration:

```bash
sudo openvpn3 config-import --config $HOME/.openvpn3/client.conf --name <ConnectionName> --persistent
```

## Step 2 – Connecting to a VPN Server with OpenVPN Client

Once you’ve configured a connection, you can easily connect to a VPN server. All you need to do is type in the following command in the terminal window:

```bash
sudosudo openvpn3 session-start --config <ConnectionName>
```

This will start the OpenVPN Client and you’ll be prompted to enter your VPN username and password. Once you’ve done that, you’ll be connected to the VPN server.

### List the open connection(s)

```bash
sudo openvpn3 sessions-list
```

### Close the connection(s)

```bash
sudo openvpn3 session-manage --session-path <SessionPath> --disconnect
```

## Step 3 – Verify Connection

Once your client machine successfully established the VPN connection, A new virtual interface is created on your system named tun0. The OpenVPN server assigned an IP address to this interface based on server configuration. This IP will be in the same network as the VPN server. To view IP address of this virtual interface, type:

```bash
ip a show tun0 
```

Output:

```bash
4: tun0:  mtu 1500 qdisc fq_codel state UNKNOWN group default qlen 100
    link/none 
    inet 10.8.0.6 peer 10.8.0.5/32 scope global tun0
       valid_lft forever preferred_lft forever
    inet6 fe80::7226:57b1:f101:313b/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever
```

## Step 4 - automatic connection at boot

To make an automatic connection at boot time, copy the configuration file in `/etc/openvpn/autoload`, and create a `.autoload` file as follows:

```bash
sudo cp $HOME/.openvpn3/client.conf /etc/openvpn/autoload
sudo vi /etc/openvpn/autoload/client.autoload
```

```bash
{
"name": "homelab",
"autostart": true,
"user-auth": {
  "autologin": true,
  "username": "username",
  "password": "password"
},
"tunnel": {
  "persist": true
}
}
```

Check the autoload utility:

```bash
/usr/sbin/openvpn3-autoload --directory /etc/openvpn3/autoload
```

Then, setup the openvpn3 service daemon:

```bash
sudo systemctl daemon-reload
sudo systemctl enable openvpn3-autoload
sudo systemctl start openvpn3-autoload
sudo systemctl status openvpn3-autoload
```

## Troubleshooting Tips for Installing OpenVPN Client on Ubuntu

If you’re having trouble installing OpenVPN Client on Ubuntu, here are a few troubleshooting tips that might help:

- Make sure that your OpenVPN configuration file is correct. Double-check it for any typos or errors and make sure all the settings are correct.
- Make sure that your internet connection is stable and fast. If your connection is slow or unreliable, it may cause issues with the installation process.
- If you’re still having trouble, try reinstalling the OpenVPN package. You can do this by typing in the following command in the terminal window: sudo apt --purge remove openvpn followed by sudo apt install openvpn.
- If you still can’t get it to work, try using a different OpenVPN client. There are several alternatives available such as OpenConnect, OpenVPN GUI, or Viscosity.
