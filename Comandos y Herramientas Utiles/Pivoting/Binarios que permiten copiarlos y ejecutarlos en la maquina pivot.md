https://github.com/andrew-d/static-binaries

De aqui podemos descargar binario y subirlos a la maquina

Ej Descargamos un binario de nmap
https://github.com/andrew-d/static-binaries/tree/master/binaries/linux/x86_64

Lo copiamos a la maquina y lo ejecutamos con las opciones basicas para ver los puertos disponibles en la otra red
```
bash-5.0# ./nmap -Pn -sS 10.10.0.129

PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 00:0C:29:BB:27:78 (Unknown)

```