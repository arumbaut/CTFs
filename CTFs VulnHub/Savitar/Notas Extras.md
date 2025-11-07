Este es el que funciona despues de hacer las pruebas con los 2
 de aqui vemos que se nos conecta una session a nuestro puerto de nc

### `$(...)`

Esto es **expansión de comandos** en Bash. Significa: _ejecuta el comando dentro de los paréntesis y reemplázalo por su salida_.
```
docker volume rm $(docker volume ls -q) este comando se convierte en 

docker volume rm vol1 vol2 vol3 esto sirve para borrar todos los volumenes

```

Otras variante 
```
docker rm $(docker ps -a -q) --force  borra todos los contenedores

docker rmi $(docker images -q) borra todos las imagenes
```
