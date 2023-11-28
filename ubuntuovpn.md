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
sudo apt update && sudo apt install openvpn -y 
```

Once the package is installed, you’ll need to create a configuration file. To do this, type in the following command in the terminal window:

```bash
sudo nano /etc/openvpn/client.conf 
```

This will open a text editor where you can paste the configuration file (opvn) from your VPN provider.

Once you’ve pasted the configuration file, save and exit the text editor by pressing Ctrl+X followed by Y and then Enter.

## Step 2 – Connecting to a VPN Server with OpenVPN Client

Once you’ve installed OpenVPN Client on Ubuntu, you can easily connect to a VPN server. All you need to do is type in the following command in the terminal window:

```bash
sudo openvpn --config /etc/openvpn/client.conf 
```

This will start the OpenVPN Client and you’ll be prompted to enter your VPN username and password. Once you’ve done that, you’ll be connected to the VPN server.

Output:

```bash
Sat Feb 29 15:39:18 2020 TCP/UDP: Preserving recently used remote address: [AF_INET]69.87.218.145:1194
Sat Feb 29 15:39:18 2020 Socket Buffers: R=[212992->212992] S=[212992->212992]
Sat Feb 29 15:39:18 2020 UDP link local: (not bound)
Sat Feb 29 15:39:18 2020 UDP link remote: [AF_INET]69.87.218.145:1194
Sat Feb 29 15:39:18 2020 TLS: Initial packet from [AF_INET]69.87.218.145:1194, sid=6d27e1cb 524bd8cd
Sat Feb 29 15:39:18 2020 VERIFY OK: depth=1, CN=Easy-RSA CA
Sat Feb 29 15:39:18 2020 VERIFY OK: depth=0, CN=tecadmin-server
Sat Feb 29 15:39:18 2020 Control Channel: TLSv1.3, cipher TLSv1.3 TLS_AES_256_GCM_SHA384, 2048 bit RSA
Sat Feb 29 15:39:18 2020 [tecadmin-server] Peer Connection Initiated with [AF_INET]69.87.218.145:1194
Sat Feb 29 15:39:19 2020 SENT CONTROL [tecadmin-server]: 'PUSH_REQUEST' (status=1)
Sat Feb 29 15:39:19 2020 PUSH: Received control message: 'PUSH_REPLY,redirect-gateway def1,dhcp-option DNS 208.67.222.222,dhcp-option DNS 208.67.220.220,route 10.8.0.1,topology net30,ping 20,ping-restart 60,ifconfig 10.8.0.6 10.8.0.5,peer-id 0,cipher AES-256-GCM'
Sat Feb 29 15:39:19 2020 OPTIONS IMPORT: timers and/or timeouts modified
Sat Feb 29 15:39:19 2020 OPTIONS IMPORT: --ifconfig/up options modified
Sat Feb 29 15:39:19 2020 OPTIONS IMPORT: route options modified
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

You can also check the OpenVPN server log to verify the connection status:

```bash
sudo tail -f /var/log/openvpn.log 
```

You should see the following output:

Output:

```bash
Fri Feb 21 15:39:18 2020 45.58.34.83:37445 Control Channel: TLSv1.3, cipher TLSv1.3 TLS_AES_256_GCM_SHA384, 2048 bit RSA
Sat Feb 29 15:41:18 2020 45.58.34.83:37445 [client] Peer Connection Initiated with [AF_INET]45.58.34.83:37445
Sat Feb 29 15:41:18 2020 client/45.58.34.83:37445 MULTI_sva: pool returned IPv4=10.8.0.6, IPv6=(Not enabled)
Sat Feb 29 15:41:18 2020 client/45.58.34.83:37445 MULTI: Learn: 10.8.0.6 -> client/45.58.34.83:37445
Sat Feb 29 15:41:18 2020 client/45.58.34.83:37445 MULTI: primary virtual IP for client/45.58.34.83:37445: 10.8.0.6
Sat Feb 29 15:41:19 2020 client/45.58.34.83:37445 PUSH: Received control message: 'PUSH_REQUEST'
Sat Feb 29 15:41:19 2020 client/45.58.34.83:37445 SENT CONTROL [client]: 'PUSH_REPLY,redirect-gateway def1,dhcp-option DNS 208.67.222.222,dhcp-option DNS 208.67.220.220,route 10.8.0.1,topology net30,ping 20,ping-restart 60,ifconfig 10.8.0.6 10.8.0.5,peer-id 0,cipher AES-256-GCM' (status=1)
Sat Feb 29 15:41:19 2020 client/45.58.34.83:37445 Data Channel: using negotiated cipher 'AES-256-GCM'
Sat Feb 29 15:41:19 2020 client/45.58.34.83:37445 Outgoing Data Channel: Cipher 'AES-256-GCM' initialized with 256 bit key
Sat Feb 29 15:41:19 2020 client/45.58.34.83:37445 Incoming Data Channel: Cipher 'AES-256-GCM' initialized with 256 bit key
```

## Troubleshooting Tips for Installing OpenVPN Client on Ubuntu

If you’re having trouble installing OpenVPN Client on Ubuntu, here are a few troubleshooting tips that might help:

- Make sure that your OpenVPN configuration file is correct. Double-check it for any typos or errors and make sure all the settings are correct.
- Make sure that your internet connection is stable and fast. If your connection is slow or unreliable, it may cause issues with the installation process.
- If you’re still having trouble, try reinstalling the OpenVPN package. You can do this by typing in the following command in the terminal window: sudo apt --purge remove openvpn followed by sudo apt install openvpn.
- If you still can’t get it to work, try using a different OpenVPN client. There are several alternatives available such as OpenConnect, OpenVPN GUI, or Viscosity.