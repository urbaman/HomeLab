# Dell Openmanage Installation

We will keep Openmanage to 10300 version, actually last compatible to Debian 11 (Proxmox)

apt-get install ca-certificates apt-transport-https

echo "deb https://linux.dell.com/repo/community/openmanage/10300/focal foacal main" | sudo tee -a /etc/apt/sources.list.d/omsa.list

wget https://linux.dell.com/repo/pgp_pubkeys/0x1285491434D8786F.asc
apt-key add 0x1285491434D8786F.asc

wget http://archive.ubuntu.com/ubuntu/pool/universe/o/openwsman/libwsman-curl-client-transport1_2.6.5-0ubuntu5_amd64.deb
wget http://archive.ubuntu.com/ubuntu/pool/universe/o/openwsman/libwsman-client4_2.6.5-0ubuntu5_amd64.deb
wget http://archive.ubuntu.com/ubuntu/pool/universe/o/openwsman/libwsman1_2.6.5-0ubuntu5_amd64.deb
wget http://archive.ubuntu.com/ubuntu/pool/universe/o/openwsman/libwsman-server1_2.6.5-0ubuntu5_amd64.deb
wget http://archive.ubuntu.com/ubuntu/pool/universe/s/sblim-sfcc/libcimcclient0_2.2.8-0ubuntu2_amd64.deb
wget http://archive.ubuntu.com/ubuntu/pool/universe/o/openwsman/openwsman_2.6.5-0ubuntu5_amd64.deb
wget http://archive.ubuntu.com/ubuntu/pool/multiverse/c/cim-schema/cim-schema_2.48.0-0ubuntu1_all.deb
wget http://archive.ubuntu.com/ubuntu/pool/universe/s/sblim-sfc-common/libsfcutil0_1.0.1-0ubuntu4_amd64.deb
wget http://archive.ubuntu.com/ubuntu/pool/multiverse/s/sblim-sfcb/sfcb_1.4.9-0ubuntu5_amd64.deb
wget http://archive.ubuntu.com/ubuntu/pool/universe/s/sblim-cmpi-devel/libcmpicppimpl0_2.0.3-0ubuntu3_amd64.deb

dpkg -i libwsman-curl-client-transport1_2.6.5-0ubuntu5_amd64.deb
dpkg -i libwsman-client4_2.6.5-0ubuntu5_amd64.deb
dpkg -i libwsman1_2.6.5-0ubuntu5_amd64.deb
dpkg -i libwsman-server1_2.6.5-0ubuntu5_amd64.deb
dpkg -i libcimcclient0_2.2.8-0ubuntu2_amd64.deb
dpkg -i openwsman_2.6.5-0ubuntu5_amd64.deb
dpkg -i cim-schema_2.48.0-0ubuntu1_all.deb
dpkg -i libsfcutil0_1.0.1-0ubuntu4_amd64.deb
dpkg -i sfcb_1.4.9-0ubuntu5_amd64.deb
dpkg -i libcmpicppimpl0_2.0.3-0ubuntu3_amd64.deb

apt update
apt install srvadmin-all libncurses5 libxslt-dev libgd-tools gpm lm-sensors snmptrapd fancontrol i2c-tools

