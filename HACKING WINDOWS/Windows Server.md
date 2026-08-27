Componentes internos clave
==LSASS:== Valida credenciales y guarda secretos (hashes, tickets) en memoria. Objetivo central de Mimikatz (Clase 7).
==SAM:== Base de datos de cuentas locales del equipo.
==Registro:== Configuración del sistema; contiene hives como SAM y SYSTEM.
==SCM:== Service Control Manager: gestiona los servicios; su abuso permite escalar a SYSTEM.


Componente Descripción
Domain Controller (DC) Servidor que almacena NTDS.dit y valida la autenticación.
Dominio Límite lógico de administración. Ej.: corp.local
Árbol / Bosque Jerarquías de dominios; el bosque es el límite de seguridad máximo.
OU Contenedor lógico para organizar y delegar.
GPO Políticas aplicadas a usuarios y equipos (almacenadas en SYSVOL).
Objetos Usuarios, equipos, grupos y cuentas de servicio; cada uno con SID 