# Prometheus snmp exporter for Arista Switch and Dell IDRAC

Use the given snmp.yml and the given dashboards.

Switch dashboard: use the given json

```bash
sed -rn 's/.*"(expr)": (.*)/\1 \2/p' prova.json
```
