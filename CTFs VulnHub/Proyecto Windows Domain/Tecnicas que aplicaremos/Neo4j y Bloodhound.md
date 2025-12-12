Herramientas para la enumeracion de dominios y nos da informacion de vulneravilidades a explotar

```
apt instal neo4j bloodhound


neo4j console

Cambiar password en neo4j entrando a la web que nos sugiere

Levantamos bloodhound

bloodhound &>/dev/null &

disown

```

Ahora devemos subr un zip al bloodhound
Que o haremos de la siguiente manera
Nos descargamos el archivo SharpHound.ps1 del repo de github

Nos conectamos con psexec a la maquina del dominio preferiblemente un DC
y nos llevamos el SharpHound.ps1 a la maquina objetivo

En la maquina objetivo ejecutamos el siuiente comando desde donde tengamos el SharpHound.ps1 que lo que vamos a ejecutar esta dentro del fichero asi que podemos grepearlo para localizar la funcionalidad grep -i 'collection'

```
Invoke-BloodHound -CollectionMethods All

 Esto genera un zip que nos lo descargamos a la maquina attacante donde estan ejecutandose neo4j bloodhound y lo subimos a bloodhound
```




