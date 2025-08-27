# Introducción:
En está sección vamos a colocar todas aquellas formas o vias de realizar shells, en diferentes lenguajes y formas:

## Lenguaje C:
- Fichero que abre una sesión de shell en la terminal actual:
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main() {
    setuid(0); //Trata de forzar que el uid sea 0, es decir el de root (requiere privilegios)
    setgid(0); //Trata de forzar que el grupo sea 0, es decir el de root (requiere privilegios)
    execl("/bin/bash", "bash", NULL); //Sustituye el PID actual y corre el fichero "/bin/bash" llamandolo "bash", sí es una ruta relativa ira a leer "PATH"
    return 0;
}
```
- Ejecución de comando sin abrir shell:
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main() {
    setuid(0); //Trata de forzar que el uid sea 0, es decir el de root (requiere privilegios)
    setgid(0); //Trata de forzar que el grupo sea 0, es decir el de root (requiere privilegios)
    execl("/bin/bash", "bash", "-c", "echo hi", (char *)NULL); //Sustituye el PID actual y corre el comando con argumentos
    return 0;
}
```
- Compilarlo:
```
gcc file.c -o exec.elf
```

