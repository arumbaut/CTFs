Conectar con null session
```
rpcclient -U "" 192.168.1.80 -N
```


Ejecutar comandos de irectamente aqui estamos filtrando para ver alguna opcion sid
```
rpcclient -U "" 192.168.1.80 -N -c "--help" | grep sid

Para ver los sid 
rpcclient -U "" 192.168.1.80 -N -c "lsaenumsid"
```