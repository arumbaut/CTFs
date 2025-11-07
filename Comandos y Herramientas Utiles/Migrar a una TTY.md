Para tener un TTY interactiva donde nos brinde mejores funcionalidade  que las 
que nos brinda la coneccion inicial
Truco clásico de Unix/Linux que **abre una nueva shell "limpia"** usando el comando `script`, pero **sin guardar un archivo de registro**

```
 script /dev/null -c bash 

```
 precionamos Ctrl+z para pausar nc y ejecutamos el comando 
```
 stty raw -echo; fg
```
 
 y luego reseteamos la terminal con 
```
 reset xterm 
```
 
 esto nos permitira precionar Ctrl+c sin sacarnos del la terminal remota 
 luego revisamos
```
echo $TERM 
para cambiar su valor a xterm  

export TERM=xterm
esto nos permitira utilizar Ctrl+l

Revisaremos la SHELl y la modificamos de ser necesario
echo $SHELL

Cambiarla
export SHELL=/bin/bash
```
 
 Ahora modificaremos las dimenciones de la tty para trabajar mejor
 Revisamos el tamanno del **tty** 
```
 stty size  
```
 y revisamos el de nuestra consola para agregarlo a la de la maquina objetivo como:
```
stty rows 43 columns 184 
```
esos son los valores de nuestra consola personal