[[Escenario de ejemplo Chisel y Socat]]

Nuestro Kali sera nuestro servidor de Chisel.  Aqui ponemos a chisel a la escucha para crear un tunel por el puerto 1234

```
./chisel server --reverse -port 1234
```


Maquina1  la que utilizaremos com pivot para poder ver los puertos de la tercera maquina de la cual no formamos parte de su red. Esto se hace un vez comprometida la Mauquina 1. Nos copiamos el chisel a la maquina 1 y lo conectamos al servidor como cliente para hacer el port forwarding

```
./chisel client server:port R:port #para un puerto en especifico

./chisel client server:port R:socks #para todos los puertos

Ej

./chisel client server:port R:80 veriamos solo el 80 de la maquina 2

./chisel client server:port R:socks
```

Nota para poder acceder a este tunel creado hay que indicarselo a las aplicaciones mediante un proxy que rediria el trafico a este tunel
Se puede hacer en el navegador con FoxyProxy o para la consola utilizar proxychains

```
nano /etc/proxychains4.conf 

Agregamos
 
socks5  127.0.0.1 1080

1080 Puerto por defecto, pero podemos indicar que sea por otro ciando creamos el tunel sock

./chisel client 192.168.1.108:1234 R:5000:socks 

En ete caso tendriamos que poner en el proxy 

socks5  127.0.0.1 5000

```

Esto sirve para tener visibilidad de la maquina a la queremos llegar pero no ocurre en sentido contrario por lo que si quisieramos hacer una ReverseShell no podriamos para esto es necesrio uilizar otra herramienta llamada 