# ACTIVE DIRECTORY STRUCTURE:
```
FOREST:
|INLANEFREIGHT.LOCAL/ (TREE)
|├── ADMIN.INLANEFREIGHT.LOCAL
|│   ├── GPOs
|│   └── OU (Unidades organizacionales)
|│       └── EMPLOYEES
|│           ├── COMPUTERS
|│           │   └── FILE01
|│           ├── GROUPS
|│           │   └── HQ Staff
|│           └── USERS
|│               └── barbara.jones
|├── CORP.INLANEFREIGHT.LOCAL
|└── DEV.INLANEFREIGHT.LOCAL
|
|INLANAMER.LOCAL/ (TREE)
 └── ADMIN.INLANAMER.LOCAL 
```
## Termonology:
- GUID: 
  - Valor de 128-bit que sirve como identificador unico para los objetos creados en el AD, como los users, groups, computers, domain, domain controller.
- Security Identifier (SID): 
  - Valor tipo GUID unico que se asocian al usuario con una lista de permisos, este ID tiene una relación uno a uno, no se puede asignar a varios objetos o viceversa, es unico.
  - Más sin embargo existen los Well-known SIDs que son "genericos" y se usan para temas generales, como:
   - S-1-5-18	    SYSTEM
   - S-1-5-32-544	Administrators (grupo local)
   - S-1-5-11	    Authenticated Users
  - Ejemplo:
   - S-1-5-21-3623811015-3361044348-30300820-1013
    - S: Indica que es un SID.
    - 1: Versión del SID.
    - 5: Autoridad de identificación (5 = NT Authority).
    - 21-...: Dominio o identificador único del emisor.
    - 1013: RID (Relative Identifier), identifica al objeto dentro del dominio.
 - DN:
  - Describe la estructura a la pertene un objeto.
  - cn=bjones, ou=IT, ou=Employees, dc=inlanefreight, dc=local
 - sAMAccountName
  - Username of the account
 - Read-Only Domain Controller (RODC)
  - AD de solo lectura que no permite modificaciones
 - Replication:
  - Proceso de replicacion de los cambios en el AD encargado por el servicio Knowledge Consistency Checker (KCC)














