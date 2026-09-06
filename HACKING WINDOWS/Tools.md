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
