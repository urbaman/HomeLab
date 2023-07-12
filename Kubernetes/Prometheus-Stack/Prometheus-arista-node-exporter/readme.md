# Arista EOS

The third device in a row and the third one, where we can setup the Prometheus exporter on top of. The approach is very similar to the first two: Arista EOS has Linux under the hood; therefore, we are need to download Prometheus Node Exporter binary, put into the corresponding directory and then to create a service.

## Download Prometheus Exporter to Arista EOS

Unlike Cisco NX-OS, you don’t need to create an LXC or create any other container. All you need is to go inside the Bash CLI:


```bash
arista-eos# bash

Arista Networks EOS shell

[zthnat@arista-eos ~]$
```

Once you are done, download the Prometheus Node Exporter using the same approach. The only difference in this setup compared to the two previous, we don’t use management VRF; thus, you can omit “ip netns exec mgmt” entirely for all the commands:

```bash
[zthnat@arista-eos ~]$ wget https://github.com/prometheus/node_exporter/releases/download/v1.3.1/node_exporter-1.3.1.linux-amd64.tar.gz
HTTP request sent, awaiting response... 200 OK
Length: 9033415 (8.6M) [application/octet-stream]
Saving to: 'node_exporter-1.3.1.linux-amd64.tar.gz'

100%[=============================================================================================================================&gt;] 9,033,415   8.00MB/s   in 1.1s  

2022-07-31 19:15:24 (8.00 MB/s) - 'node_exporter-1.3.1.linux-amd64.tar.gz' saved [9033415/9033415]
```

Following the same approach explained before for NVIDIA Cumulus Linux and Cisco NX-OS, unpack the downlodaed file:

```bash
[zthnat@arista-eos ~]$ tar -xf node_exporter-1.3.1.linux-amd64.tar.gz
[zthnat@arista-eos ~]$ ls -l
total 8824
drwxr-xr-x 2 zthnat eosadmin     100 Dec  5  2021 node_exporter-1.3.1.linux-amd64
-rw-r--r-- 1 zthnat eosadmin 9033415 Dec  8  2021 node_exporter-1.3.1.linux-amd64.tar.gz
```

And verify its content:

```bash
[zthnat@arista-eos ~]$ ls -l node_exporter-1.3.1.linux-amd64/
total 17820
-rw-r--r-- 1 zthnat eosadmin    11357 Dec  5  2021 LICENSE
-rw-r--r-- 1 zthnat eosadmin      463 Dec  5  2021 NOTICE
-rwxr-xr-x 1 zthnat eosadmin 18228926 Dec  5  2021 node_exporter
```

By this time you should already have figured out that we are using the same Prometheus node exporter for all our switches. However, if you have base different to x86_64 (e.g., arm32 or arm64 based), you may need to use a different Prometehus Node Exporter binary.

## Create a Service with Prometheus Node Exporter at Arista EOS

Put the Prometheus Node Exporter application in the same directory you did for Cisco NX-OS:

```bash
[zthnat@arista-eos ~]$ sudo cp node_exporter-1.3.1.linux-amd64/node_exporter /usr/local/bin/.
```

And create the service afterwards. Remeber, however, that this time we create it in the default VRF rather than in the management one:

```bash
[zthnat@arista-eos ~]$ sudo tee /etc/systemd/system/node_exporter-management.service << EOF
[Unit]
Description=Node Exporter
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
EOF
```

The aforementioned fact of launching the node exporter in the default VRF reflects itsels in the ExecStart key, where we specify only the path to the node exporter without the “ip netns exec vrf_name” part.

The next step is to launch the service re-reading the content of the directory with services beforehand:

```bash
[zthnat@arista-eos ~]$ sudo systemctl daemon-reload
[zthnat@arista-eos ~]$ sudo systemctl start node_exporter-management.service
[zthnat@arista-eos ~]$ sudo systemctl status node_exporter-management.service
* node_exporter-management.service - Node Exporter
   Loaded: loaded (/etc/systemd/system/node_exporter-management.service; disabled; vendor preset: disabled)
   Active: active (running) since Sun 2022-07-31 19:18:23 UTC; 4s ago
 Main PID: 28589 (node_exporter)
   CGroup: /system.slice/node_exporter-management.service
           `-28589 /usr/local/bin/node_exporter
```

If you service is up and running, move to the next step.

## Verification of Prometheus Node Exporter on Arista EOS

The validation steps are exactly the same as you have already seen two times:

Check if the port is opened
Validate the operation locally from the network funciton using curl.
Validate the operation of the Prometheus Node Exporter remotely from the Prometheus server.
The first step is achieved using the same command as before:

```bash
[zthnat@arista-eos ~]$ ss -tnldp | grep 9100
tcp    LISTEN     0      1024      *:9100                  *:*
```

Once you validate that the port is being listened, you can check the operation of the service locally:

```bash
[zthnat@arista-eos ~]$ curl -X GET http://localhost:9100/metrics
# HELP go_gc_duration_seconds A summary of the pause duration of garbage collection cycles.
# TYPE go_gc_duration_seconds summary
go_gc_duration_seconds{quantile="0"} 0
go_gc_duration_seconds{quantile="0.25"} 0
go_gc_duration_seconds{quantile="0.5"} 0
go_gc_duration_seconds{quantile="0.75"} 0
go_gc_duration_seconds{quantile="1"} 0
go_gc_duration_seconds_sum 0
go_gc_duration_seconds_count 0
# HELP go_goroutines Number of goroutines that currently exist.
# TYPE go_goroutines gauge
go_goroutines 7
# HELP go_info Information about the Go environment.
# TYPE go_info gauge
go_info{version="go1.17.3"} 1
# HELP go_memstats_alloc_bytes Number of bytes allocated and still in use.
# TYPE go_memstats_alloc_bytes gauge
go_memstats_alloc_bytes 1.426936e+06
# HELP go_memstats_alloc_bytes_total Total number of bytes allocated, even if freed.
# TYPE go_memstats_alloc_bytes_total counter
go_memstats_alloc_bytes_total 1.426936e+06
# HELP go_memstats_buck_hash_sys_bytes Number of bytes used by the profiling bucket hash table.
# TYPE go_memstats_buck_hash_sys_bytes gauge
go_memstats_buck_hash_sys_bytes 1.445975e+06
# HELP go_memstats_frees_total Total number of frees.
# TYPE go_memstats_frees_total counter
!
! FURTHER OUTPUT IS TRUNCATED FOR BREVITY
```

So far everything looked identical. However, now there is a bit of route de-tour. By default the port for the Prometheus node exporter (9100/tcp) is not opened in the iptables in Arista EOS. As such, if you don’t open it, you won’t be able to reach it. Therefore, you should open it:

```bash
[zthnat@arista-eos ~]$ sudo iptables -I INPUT 1 -p tcp --dport 9100 -j ACCEPT
```

Once this operation is completed, you can test the operation of Prometheus Node Exporter from the remote server:

```bash
# curl -X GET http://192.168.101.13:9100/metrics
# HELP go_gc_duration_seconds A summary of the pause duration of garbage collection cycles.
# TYPE go_gc_duration_seconds summary
go_gc_duration_seconds{quantile="0"} 0
go_gc_duration_seconds{quantile="0.25"} 0
go_gc_duration_seconds{quantile="0.5"} 0
go_gc_duration_seconds{quantile="0.75"} 0
go_gc_duration_seconds{quantile="1"} 0
go_gc_duration_seconds_sum 0
go_gc_duration_seconds_count 0
# HELP go_goroutines Number of goroutines that currently exist.
# TYPE go_goroutines gauge
go_goroutines 7
# HELP go_info Information about the Go environment.
# TYPE go_info gauge
go_info{version="go1.17.3"} 1
# HELP go_memstats_alloc_bytes Number of bytes allocated and still in use.
# TYPE go_memstats_alloc_bytes gauge
go_memstats_alloc_bytes 3.05076e+06
!
! FURTHER OUTPUT IS TRUNCATED FOR BREVITY
```
