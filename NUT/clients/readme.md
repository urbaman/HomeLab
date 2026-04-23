# Clients

## Proxmox clients

Proxmox clients connect to NUT server, get the notifications and begin shutting down VM/CT in the provided order (each VM/CT, boot/shutdown order).

### /etc/nut/upsmon.conf

```bash
MONITOR ups@<IP_RPI4> 1 upsmon_user password secondary
SHUTDOWNCMD "/sbin/shutdown -h +0"
```

### BIOS

Set `Always On` in the BIOS power settings

## PfSense

Set FINAL_DELAY properly in advanced settings

###

Set `Always On` in the BIOS power settings

## Master

### /etc/nut/ups.conf

Add the `ignore = bypass` line to the UPS definition if you need the clients not to react to the UPS being in Eco Mode (Bypass)

### /etc/nut/upsmon.conf

```bash
MONITOR ups@localhost 1 upsmon_user password master
# Invece di spegnere subito, passiamo i messaggi allo scheduler
NOTIFYFLAG ONBATT SYSLOG+WALL+EXEC
NOTIFYFLAG ONLINE SYSLOG+WALL+EXEC
NOTIFYFLAG LOWBATT SYSLOG+WALL+EXEC

# Definiamo lo script che gestirà gli eventi
NOTIFYCMD /sbin/upssched

# Il comando di shutdown del master viene eseguito solo DOPO che i client sono disconnessi
SHUTDOWNCMD "/sbin/shutdown -h +0"
```

### /etc/nut/upssched.conf

```bash
CMDSCRIPT /usr/bin/upssched-cmd
PIPEFN /var/run/nut/upssched.pipe
LOCKFN /var/run/nut/upssched.lock

# Se va a batteria, avvia un timer di 15 minuti chiamato 'shutdown-timer'
AT ONBATT * START-TIMER shutdown-timer 900 # 1200 for 20 minutes

# Se torna la corrente, annulla il timer
AT ONLINE * CANCEL-TIMER shutdown-timer

# Se la batteria è CRITICA prima dei 15 min, spegni subito (Sicurezza)
AT LOWBATT * EXECUTE forced-shutdown
```

### /usr/bin/upssched-cmd

Create the Telegram bot, get the Bot Token and the Chat ID

Create the file and make it executable (`chmod +x`): 

```bash
#!/bin/bash

TOKEN="TUO_TOKEN_TELEGRAM"
CHAT_ID="TUO_CHAT_ID"

send_telegram() {
    curl -s "https://api.telegram.org/bot$TOKEN/sendMessage" -d "chat_id=$CHAT_ID&text=$1"
}

case $1 in
    shutdown-timer)
        send_telegram "⚠️ UPS a batteria da 15 minuti! Avvio procedura di spegnimento del laboratorio..."
        # Invia il comando FSD (Forced Shutdown) a tutti i client (Proxmox, pfSense)
        /sbin/upsmon -c fsd
        ;;
    forced-shutdown)
        send_telegram "🚨 BATTERIA SCARICA! Spegnimento immediato di emergenza!"
        /sbin/upsmon -c fsd
        ;;
    *)
        logger -t upssched-cmd "Evento non gestito: $1"
        ;;
esac
```

### /etc/nut/upsserv.conf

Check that command `upsdrvctl shutdown` is active
