Componentes internos clave
==LSASS:== Valida credenciales y guarda secretos (hashes, tickets) en memoria. Objetivo central de Mimikatz (Clase 7).
==SAM:== Base de datos de cuentas locales del equipo.
==Registro:== Configuración del sistema; contiene hives como SAM y SYSTEM.
==SCM:== Service Control Manager: gestiona los servicios; su abuso permite escalar a SYSTEM.
==WINLOGON:== Control de sesion 

Componente Descripción
Domain Controller (DC) Servidor que almacena NTDS.dit y valida la autenticación.
Dominio Límite lógico de administración. Ej.: corp.local
Árbol / Bosque Jerarquías de dominios; el bosque es el límite de seguridad máximo.
OU Contenedor lógico para organizar y delegar.
GPO Políticas aplicadas a usuarios y equipos (almacenadas en SYSVOL).
Objetos Usuarios, equipos, grupos y cuentas de servicio; cada uno con SID 

| Puerto | Servicio           | Importancia            |
| -----: | ------------------ | ---------------------- |
|     53 | DNS                | Resolución del dominio |
|     88 | Kerberos           | Autenticación AD       |
|    135 | RPC                | Servicios Windows      |
|    139 | NetBIOS            | SMB antiguo            |
|    389 | LDAP               | Directorio             |
|    445 | SMB                | Shares / enumeración   |
|    464 | Kerberos password  | Cambio de contraseña   |
|    636 | LDAPS              | LDAP sobre TLS         |
|   3268 | Global Catalog     | AD                     |
|   3269 | Global Catalog SSL | AD                     |