Ecaneo de puertos por cada host
```
#!/bin/bash

for i in $(seq 1 254); do
	for port in 21 22 25 443 445 8080 5985 88; do
		timeout 1 bash -c "echo '' > /dev/tcp/10.10.0.$i/$port" 2> /dev/null && echo "[+] Host 10.10.0.$i - Port $port ACTIVO" &
	done
done; wait
```

Escaneo de todos los puertos en un host objetivo
```
for port in $(seq 1 65535); do
		timeout 1 bash -c "echo '' > /dev/tcp/10.10.0.129/$port" 2> /dev/null && echo "[+] Host 10.10.0.129 - Port $port ACTIVO" &
done; wait
```