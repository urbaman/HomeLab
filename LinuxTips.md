# Lots of Linux Tips

## Purge previously uninstalled packages

```bash
sudo apt purge `dpkg --list | grep ^rc | awk '{ print $2; }'`
```
