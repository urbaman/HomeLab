# Tentativo

```
sudo apt install zfsutils-linux
```

```
sudo fdisk -l
```

```
sudo zpool create new-pool mirror /dev/sdb /dev/sdc
```

```
sudo zpool create HDD5T mirror /dev/sdb /dev/sdc
sudo zpool create SDD5T mirror /dev/sdd /dev/sde
```

```
sudo apt install glusterfs-server
sudo systemctl enable glusterd
sudo systemctl start glusterd
```

```
sudo gluster peer probe gluster2.urbaman.it
sudo gluster peer probe gluster3.urbaman.it
sudo gluster peer status
```

```
sudo apt install nfs-ganesha-gluster
sudo mv /etc/ganesha/ganesha.conf /etc/ganesha/ganesha.conf.org
sudo vi /etc/ganesha/ganesha.conf
```

```
# create new
NFS_CORE_PARAM {
    # possible to mount with NFSv3 to NFSv4 Pseudo path
    mount_path_pseudo = true;
    # NFS protocol
    Protocols = 3,4;
}
EXPORT_DEFAULTS {
    # default access mode
    Access_Type = RW;
}
EXPORT {
    # uniq ID
    Export_Id = 101;
    # mount path of Gluster Volume
    Path = "/HDD5T";
    FSAL {
        # any name
        name = GLUSTER;
        # hostname or IP address of this Node
        hostname="gluster1.urbaman.it";
        # Gluster volume name
        volume="HDD5T";
    }
    # config for root Squash
    Squash="No_root_squash";
    # NFSv4 Pseudo path
    Pseudo="/HDD5T";
    # allowed security options
    SecType = "sys";
}
LOG {
    # default log level
    Default_Log_Level = WARN;
}
```

```
sudo systemctl restart nfs-ganesha
```
