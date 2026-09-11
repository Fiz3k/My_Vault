**==Responder==** (Capturar/autenticar credenciales mediante protocolos de red. hash NTLM
La forma más fácil de entender **Responder** es olvidarte por un momento de que es una herramienta de pentesting y pensar en ella como alguien que **se hace pasar por otro equipo dentro de una red local**.)

==John de Ripper== (Crackea hash con diccionario)

==Evil WinRM== (Conectarse remotamente a windows con winRM)

`(echo ==contenido de archivo de texto== > ==nombre del archivo==)`

|Herramienta|¿Para qué sirve?|Qué buscamos en este lab|
|---|---|---|
|**Nmap**|Descubrimiento de puertos y servicios|Detectar el DC y servicios AD|
|**NetExec (`nxc`)**|Enumeración y auditoría de Windows/AD|Usuarios, grupos, SMB, política, LDAP|
|**ldapsearch**|Consultas directas contra LDAP|Atributos y configuración de usuarios|
|**rpcclient**|Consultas mediante MS-RPC|Usuarios, grupos y políticas|
|**smbclient**|Interacción con SMB|Shares y archivos accesibles|
|**Impacket**|Suite de herramientas para protocolos Windows/AD|Kerberos, SPN, AS-REP, etc.|
|**GetUserSPNs**|Enumeración de SPN|Identificar cuentas candidatas a Kerberoasting|
|**GetNPUsers**|Consulta de cuentas sin preautenticación|Identificar/probar AS-REP Roasting|
|**nslookup / dig**|Consultas DNS|Resolver el dominio y registros Kerberos|
|**CrackMapExec/NetExec**|Enumeración remota|NetExec es el sucesor moderno de CME|


# Impacket

|`GetADUsers` / `GetUserSPNs` / `GetNPUsers`|Enumeración de usuarios, SPNs (Kerberoasting) y cuentas vulnerables a AS-REP Roasting.|
|`lookupsid` / `samrdump` / `rpcdump`|Reconocimiento vía RPC: enumeración de SIDs, base SAM y endpoints expuestos.|
|`secretsdump`|Extracción de credenciales (SAM, LSA, NTDS.dit) y ataques DCSync.|
|`psexec` / `wmiexec` / `smbexec` / `atexec` / `dcomexec`|Ejecución remota de comandos usando distintos protocolos (SMB, WMI, tareas programadas, DCOM).|
|`getTGT` / `getST` / `ticketer`|Solicitud y forja de tickets Kerberos (ataques Golden/Silver Ticket).|
|`mssqlclient` / `ntlmrelayx`|Interacción con servicios MSSQL y ataques de relay NTLM.|

**Resumen:**

Este conjunto de herramientas (parte del framework **Impacket**) cubre el ciclo típico de un ataque a Active Directory:

1. **Reconocimiento** — descubrir usuarios, SIDs y servicios expuestos sin necesidad de autenticación completa.
2. **Extracción de credenciales** — volcar hashes y secretos almacenados localmente o replicados desde el DC (DCSync).
3. **Movimiento lateral** — ejecutar comandos en máquinas remotas usando credenciales o hashes obtenidos.
4. **Abuso de Kerberos** — solicitar o falsificar tickets para escalar privilegios o mantener persistencia.
5. **Ataques a servicios específicos** — como MSSQL o relay de autenticación NTLM para pivotar dentro de la red.