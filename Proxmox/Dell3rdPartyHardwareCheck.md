# Quiet the Dell 3rd Party hardware check spinning the fans

Access the iDRAC through ssh (same IP, same user/password), and ran the command:

```bash
racadm set system.thermalsettings.ThirdPartyPCIFanResponse 0
```
