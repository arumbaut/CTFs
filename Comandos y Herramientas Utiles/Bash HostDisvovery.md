```
#!/bin/bash
for i in $(seq 1 254);do 
        timeout 1 bash -c "ping -c 1 10.10.0.$i" &> /dev/null && echo "[+]HOST 10.10.0.$i ACTIVO" || echo "[+]HOST 10.10.0.$i INACTIVO" 
done
```

Variante con hilos
```
#!/bin/bash
seq 1 254 | xargs -I {} -P 20 sh -c 'ping -c 1 -W 1 10.10.12.{} >/dev/null 2>&1 && echo "10.10.11.{} está activo"'
```

