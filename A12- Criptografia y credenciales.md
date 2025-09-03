# Introducción:
En está sección vamos a revisar como se puede realizar ataques a cifrado de contraseñas, certificados, etc.

---

## Metodología de descifrado:

1. **Identifica el formato** (KDBX, ZIP, RAR, 7z, PFX, etc.).  
2. **Extrae el material verificable** (hash/representación) con la herramienta adecuada.  
3. **Elige la estrategia de ataque** (diccionario, reglas, máscara, híbrido).  
4. **Optimiza**: usa GPU con hashcat cuando aplique, gestiona sesiones/potfiles y registra tiempos.  
5. **Valida** la contraseña, **desencripta** el artefacto y documenta el procedimiento.

> Tip: Usa diccionarios contextualizados (CEWL, patrones de la organización, variantes l33t, sufijos con años/!/?).

---

## Keepass:

### Fuerza bruta:
- **Keepass 2.x (KDBX 4.x, ~v2.36+)**
A partir de la verión 2.36 la herramienta de keepass2jhon no está soportada por lo que hay que buscar alternativas como:

```bash
git clone https://github.com/r3nt0n/keepass4brute
./keepass4brute.sh <kdbx-file> <wordlist>
```

- **KeePass 2.x (KDBX ≤ 3.x, versiones antiguas <~2.36)**
```bash
keepass2jhon fichero.kdbx > hash-kdbx
john --wordlist=/usr/share/wordlists/rockyou.txt hash-kdbx
```

### Robo en memoria:
- Keethieft: No requiere permisos de elevación y permite leer la contraseña en memoria durante la ejecución de keepass.
  - https://github.com/GhostPack/KeeThief
- Keefarce: Inyecta una DLL al proceso de keepass para el robo de la contraseña en caso de que este protegida la lectura en memoria.
  - https://github.com/denandz/KeeFarce

> **OPSEC/Detección**: EDR y reglas de seguridad detectan facilmente hoy en día inyección de DLL y lecturas de memoria.

### Robo del portapapeles:
Es comun que keepass permita copiar y pegar, por lo que es "facil" robar ese historial de portapepeles con herramientas tipo redteam:

- Covenant:
  - Plataforma completa de red team con funciones que permiten realizar lectura del portapeles.
  - [https://github.com/cobbr/Covenant/wiki/Installation-And-Startup]

---

## ZIP:
Primero identificar el **tipo de cifrado** (ZipCrypto legado vs AES/PKZIP moderno). Esto es visible en las propiedades del fichero:

### AES/PKZIP moderno:

- **Extracción de hash** con John the Ripper y ataque por diccionario:
  ```bash
  zip2john secreto.zip > zip_hash.txt
  john --wordlist=/usr/share/wordlists/rockyou.txt zip_hash.txt
  ```

- **Descompresión** tras recuperar la contraseña:
  ```bash
  unzip secreto.zip
  ```

### ZipCrypto:
Si el ZIP es **ZipCrypto** y cuentas con texto conocido del contenido, existen enfoques criptoanalíticos (p. ej., herramientas tipo *known-plaintext attack*) que pueden ser más eficientes que la fuerza bruta.

#### Abusar de Multiples ficheros o ficheros comunes: 
Sí el archivo cifrado trae varios ficheros, y entre ellos ficheros faciles de conseguir como readme.txt, license.txt, iguales o secciones que no varian, podemos usar estos ficheros para conseguir nuestro plain.bin dado que no importa sí la llave conseguida es de estos ficheros, con está llave podemos descifrar el resto del archivo.

#### Extracción sin Deflate:
Sí el archivo fue cifrado sin deflate, ejemplo de como se veria en 7zip:
<img width="821" height="251" alt="image" src="https://github.com/user-attachments/assets/c86e6520-e2ef-458d-ac93-c4bb2b3f9641" />

- **Extracción de texto plano** El primer paso es conocer una sección original del fichero a descifrar en uno nuevo, este debe estar en binario, colocamos esa pequeña sección dentro de un fichero.
<img width="891" height="161" alt="image" src="https://github.com/user-attachments/assets/28296f6a-9111-41cd-ac37-c26e2ff96420" />

- **Conseguir llave**: Nos regresara 3 llaves cortas, que usaremos adelante. 
  - https://github.com/kimci86/bkcrack
```bash
bkcrack.exe -C "Archivocompleto.zip" -c "Archivocifrado.pdf" -p plain.bin
```
<img width="2000" height="341" alt="image" src="https://github.com/user-attachments/assets/062dad7b-3f95-4b47-a07b-5b164bb2a3ee" />

- **Extracción y descifrado**:
```bash
bkcrack.exe -C "Archivocompleto.zip" -c "Archivocifrado.pdf" -k [Your] [Keys] [Here] -d "DecryptedFile.pdf"
```

#### Extracción con deflate:
Sí el archivo fue cifrado mediante compresión con deflate, hay que conocer el texto plano comprimido, 

- **Comprimir:** por lo que podemos comprimirlo con la herramienta de deflate.py:
```bash
python tools/deflate.py --level 5 < originalfile.txt > originalfile.txt.compress
```

> **--LEVEL**: Hay que hacer el deflate con varios niveles hasta encontrar el correcto, aunque por ejemplo 7zip coloca por defecto level 3.

#### Recuperar la contraseña:
La contraseña con la que fue cifrado podria ser importante o repetida en otro sistema, y es facil de extraer sí ya conoces las llaves:
```bash
bkcrack.exe -k [Your] [Keys] [Here] -r 12 ?p
```
<img width="625" height="195" alt="image" src="https://github.com/user-attachments/assets/e26e10f1-3305-42e3-8a8b-1de6cb60d9f2" />

> **Referencia de charset**: https://github.com/kimci86/bkcrack?tab=readme-ov-file#character-sets

#### Texto plano en partes:
Sí conoces poco sobre el fichero de 8 a 11 bytes, podrias complentarlo con otras secciones del fichero con el flag -x
```bash
bkcrack -c cipherfile -p plainfile -x 25 4b4f -x 30 21
```
#### 






