# Stop Ceph node

```bash
ceph osd set noscrub
ceph osd set nodeep-scrub

ceph status
```

## set cluster in maintenance mode with :

```bash
ceph osd set noout
```

## Set ceph back to working state

```bash
ceph osd unset noout
ceph osd unset noscrub
ceph osd unset nodeep-scrub

ceph status
```

# Ceph rules, maps and crushmaps

## Creating a New Class

Classes, if you don't know, are a way to categorize devices based really on their speed. You could also create classes for things like their size as well I suppose. There's a few different ways to leverage classes that are useful. For this tutorial, we're going to add an nvme class.

List OSDs and their classes:

```bash
ceph osd crush tree --show-shadow
```

Output:

```bash
ID  CLASS WEIGHT    TYPE NAME
-16  nvme   4.54498 root default~nvme
-10        37.75299     host host1
13   hdd   9.09499         osd.13
14   hdd   9.09499         osd.14
15   hdd   9.09499         osd.15
16   hdd   9.09499         osd.16
20   ssd   0.90900         osd.20     (These .90900 drives, 1TB, are the nvme disks but see how they're ssd? Note the osd number, osd.20 in this case, 20.
17   ssd   0.46500         osd.17
-7        37.74199     host host2
  8   hdd   9.09499         osd.8
  9   hdd   9.09499         osd.9
10   hdd   9.09499         osd.10
11   hdd   9.09499         osd.11
  7   ssd   0.90900         osd.7       ***
18   ssd   0.45399         osd.18
-3        39.56000     host host3
  0   hdd   9.09499         osd.0
  1   hdd   9.09499         osd.1
  2   hdd   9.09499         osd.2
  3   hdd   9.09499         osd.3
  4   ssd   0.90900         osd.4       ***
  5   ssd   0.90900         osd.5       ***
  6   ssd   0.90900         osd.6       ***
19   ssd   0.45399         osd.19
```

Note the IDs of the devices you want to change, and unset and set the new class to the device:

Unset the class ssd for device 20:

```bash
ceph osd crush rm-device-class osd.20
```

Set the class to nvme for device 20:


```bash
ceph osd crush set-device-class nvme osd.20
```

Do the above for all OSD IDs that you want to change to a particular class.

Now we need to be sure ceph doesn't default the devices, so we disable the update on start. This will allow us to modify classes as they won't be set when they come up.

Modify your local `/etc/ceph/ceph.conf` to include an osd entry:

```bash
[osd]
osd_class_update_on_start = false
```

I THINK you have to hup the osd service but I'm not really sure to be honest right now

```bash
systemctl restart ceph-osd.target
```

## Crushmap

You can see the crushmap in the UI @ Host >> Ceph >> Configuration

To understand a crushmap we need to do the following.
Export and decompile a crush map
Read and edit a crush map
Import a crush map
Rules to Pools

Get crushmap to compiled file


```bash
ceph osd getcrushmap -o crushmap.compiled
```

Decompile the crushmap


```bash
crushtool -d crushmap.compiled -o crushmap.text
```

Compile the crushmap, maintaining original compiled crushmap file (you choose how you want to manage the files)

```bash
crushtool -c crushmap.text -o crushmap.compiled.new
```

Import the new crushmap

```bash
ceph osd setcrushmap -i crushmap.compiled.new
```

Now that we know how to edit a crushmap, let's edit a crushmap!

### Creating New Rules

Alright, we have a new class created but not really doing anything. So now we need to associate a class to a rule.

Using the above crushmap instructions, decompile and edit your crush map rules. The default rule name with ID 0 is something like replicated.

I decided to delete the default rule and replace it with 3 rules. The rules are what link the classes to the pool, which we'll do in the last step. Delete the original rule id 0 and replace with these 3. This works for this example as we have 3 devices types but your discretion here what you do to your own environment. Most would create an SSD and HDD rule, for example.

```bash
# rules
rule replicated_hdd {
    id 0
    type replicated
    min_size 1
    max_size 10
    step take default class hdd
    step chooseleaf firstn 0 type host
    step emit
}
rule replicated_ssd {
    id 1
    type replicated
    min_size 1
    max_size 10
    step take default class ssd
    step chooseleaf firstn 0 type host
    step emit
}
rule replicated_nvme {
    id 2
    type replicated
    min_size 1
    max_size 10
    step take default class nvme
    step chooseleaf firstn 0 type host
    step emit
}
```

Don't forget to compile and import your new crush file!

You can test the crushfile like so, but --show-X fails for me for some reason as of this writing:

```bash
crushtool -i crushmap.compiled.new --test --show-X
```

## Create Pool

Go into Proxmox UI Host >> Ceph >> Pools

Create a pool name, like pool-ssd, and bind the new rule to that pool name. This will result in any new device that is associated with a particular class, automagically get associated with a pool based on your need. I think this is pretty damn cool. Don't you?

Your new rules are available in the pool creation.

Now you can do the following:

Upload ISOs to a pool resource and ALL nodes will be able to install from it. Woohoo!

Ceph is RBD storage only in this case and therefore doesn't support ISO Image content type, Only Container and Disk Image I believe now HA is feasible as you can push VMs to these shared pools. Which is what I'm going to test out tomorrow. Install VMs on media that works best for your need.

Enjoy!
