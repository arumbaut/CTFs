
Buscamos binarios con permisos SUID (pueden ejecutarse como root) .

https://gtfobins.github.io/

[[Binarios Utiles]]

```
find / -perm -4000 -user root 2>/dev/null
```

#### Filtrar por nombres desconocidos
```
find / -perm -4000 2>/dev/null | grep -vE "(^/usr|^/bin|^/sbin)"

find / -perm -4000 2>/dev/null | grep -v snap
-v  # No muestre la palabra que le indicamos en este caso una expr         regular
```


### Buscar capacidades especiales establecidas (capabilities)
```
getcap -r / 2>/dev/null
```

Buscar archivos modificables o escribibles por el usuario actual:
```
find / -writable ! -user $(whoami) -type f 2>/dev/null
```

directorios importantes que uno podría manipular:
```
find /etc -writable -type d 2>/dev/null
```

| Comando                                                            | Propósito                                        |
| ------------------------------------------------------------------ | ------------------------------------------------ |
| `find / -perm -4000 2>/dev/null`                                   | Búsqueda clásica de binarios SUID                |
| `find / -perm -2000 2>/dev/null`                                   | Búsqueda de binarios SGID                        |
| `getcap -r / 2>/dev/null`                                          | Revisar ficheros con capacidades peligrosas      |
| `find / -writable ! -user $USER 2>/dev/null`                       | Detectar recursos modificables peligrosos        |
| `sudo -l`                                                          | Enumerar lo que el usuario puede hacer como sudo |
| Investigar `/etc/passwd`, `/etc/shadow`, `/etc/group` (integridad) | Buscar errores de configuración                  |