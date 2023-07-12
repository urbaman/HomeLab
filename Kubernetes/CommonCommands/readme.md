# Common commands (Linux)

## Export a given yaml key repetitions to a file

```bash
sed -rn 's/.*"(expr)": (.*)/\1 \2/p' prova.json
```
