# Introducción:
En está sección vamos a colocar todas aquellas formas o vias de realizar shells, en diferentes lenguajes y formas:

## Lenguaje C:
Generar fichero en C para despues compilarlo en un ELF.
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main() {
    setuid(0);
    setgid(0);
    execl("/bin/bash", "bash", NULL);
    return 0;
}
```
- Compilarlo:
```
gcc file.c -o exec.elf
```

