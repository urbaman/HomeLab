# iDRAC Service Module(iSM) Installation Procedure

To properly install the iDRAC service module, if not installed by the Openmanage installation procedure:

```bash
wget https://linux.dell.com/repo/community/openmanage/iSM/4300/focal/pool/main/d/dcism-osc/dcism-osc_7.0.1.0_amd64.deb
wget https://linux.dell.com/repo/community/openmanage/iSM/4300/focal/pool/main/d/dcism/dcism_4.3.0.0-2781.ubuntu20_amd64.deb
```

```bash
dpkg -i dcism-osc_7.0.1.0_amd64.deb
dpkg -i dcism_4.3.0.0-2781.ubuntu20_amd64.deb
```
