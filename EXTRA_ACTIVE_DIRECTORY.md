# ACTIVE DIRECTORY STRUCTURE:

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

## Termonology:
- GUID: 
  - Valor de 128-bit que sirve como identificador unico para los objetos creados en el AD, como los users, groups, computers, domain, domain controller.
- Security Identifier (SID): 
  - Valores unicos que solo "existen una vez" para autenticar a un usuario, es decir son creados una vez y son asignados a una sesión para asociar los permisos asociados, y despues que el usuario se autentica se le quita ese SID
  - 














