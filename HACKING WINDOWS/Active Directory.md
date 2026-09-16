**AS-REP Roasting (Ataca el Paso 2):** Aprovecha las cuentas que no requieren preautenticación. El atacante pide el `AS-REP` de un usuario y, como el servidor lo entrega sin validar, se lleva el archivo para intentar descifrar la contraseña fuera de la red (_offline_).

**Kerberoasting (Ataca los Pasos 3 y 4):** El atacante ya está dentro de la red con un usuario común. Solicita un ticket (`TGS`) para un servicio específico (como una base de datos). El KDC le entrega el ticket cifrado con la clave de la cuenta de ese servicio. El atacante se lo lleva para romper la contraseña del servicio de forma _offline_.

**Pass-the-Ticket (Ataca el Paso 5):** En lugar de adivinar contraseñas, el atacante roba un ticket válido (`TGT` o `TGS`) directamente de la memoria de una computadora comprometida y lo "inyecta" en su propio equipo para suplantar la identidad de la víctima sin que esta se dé cuenta.

**Golden / Silver Ticket (Ataca todo el sistema):** Es el ataque más grave. Si el atacante toma el control total del servidor (consigue la clave de la cuenta `krbtgt` o de un servicio crítico), puede fabricar de la nada sus propios tickets falsos y permanentes, dándose a sí mismo acceso ilimitado como administrador.


Enumeracion AD 
El objetivo: DOMINIO `corp.local`

El gráfico parte del dominio objetivo (`corp.local`). Desde aquí, el atacante quiere mapear todo el entorno para encontrar vectores de ataque, escalar privilegios y moverse lateralmente.

## USUARIOS - `net user` / `GetADUsers.py`
- Listar todos los usuarios del dominio.
- Identificar **usuarios administrativos** (Administrador, admins de dominio, admins de empresa).  
- Buscar **usuarios con nombres descriptivos** (ej: `svc_sql`, `svc_backup`, `svc_exchange`) que suelen ser cuentas de servicio.

**Herramientas:**
- **`net user`**: Comando nativo de Windows (desde la máquina víctima).
- **`GetADUsers.py`**: Script de Impacket desde Kali (requiere autenticación).

- Los usuarios administrativos son objetivos de primer nivel.
- Las cuentas de servicio suelen tener **contraseñas débiles o estáticas** y son ideales para ataques de **Kerberoasting**.



EQUIPOS - `Get-DomainComputer`
- Listar todos los equipos del dominio (estaciones de trabajo, servidores, controladores de dominio).
- Identificar **sistemas operativos**, **roles** (DC, SQL, Exchange, web, archivos).
- Detectar **equipos con altos privilegios** (ej: servidores de backup, servidores de administración).

**Herramienta:**

- **`Get-DomainComputer`**: Script de PowerView (PowerShell).

- Los servidores críticos (DC, SQL, Exchange) son objetivos principales.
- Las estaciones de trabajo de administradores son objetivos de movimiento lateral.
- Equipos con sistemas operativos antiguos pueden tener vulnerabilidades.

GRUPOS - `Get-DomainGroupMember`
- Identificar miembros de grupos privilegiados:
    
    - `Domain Admins`
        
    - `Enterprise Admins`
        
    - `Administrators`
        
    - `Schema Admins`
        
    - `Server Operators`
        
    - `Backup Operators`
        
    - `Account Operators`
        

**Herramienta:**

- **`Get-DomainGroupMember`**: PowerView. 

- Encontrar **quién puede hacer qué** en el dominio.
- Identificar **anidamiento de grupos** (grupos que contienen otros grupos).
- Buscar **grupos con permisos inusuales** (ej: un grupo de usuarios normales que tiene permisos de administración local en servidores).



## CUENTAS SPN - `setspn` / `GetUserSPNs.py`

- Listar todas las cuentas con **Service Principal Names (SPN)** asociados.
- Identificar **qué servicios se ejecutan** y **qué cuentas los ejecutan**.

**Herramientas:**

- **`setspn`**: Comando nativo de Windows.
- **`GetUserSPNs.py`**: Script de Impacket desde Kali.

- Todas las cuentas con SPN son vulnerables a **Kerberoasting**.
- El atacante solicita tickets de servicio para cada SPN, los descarga y trata de descifrar sus contraseñas fuera de línea.
- Si alguna contraseña es débil, el atacante obtiene la cuenta de servicio.

SHARES SMB - `smbmap` / `smbclient`

**¿Qué se busca?**

- Listar los recursos compartidos SMB (carpetas compartidas) en los servidores
- Identificar **compartidos accesibles** con las credenciales actuales.
- Buscar **archivos sensibles** (scripts de backup, archivos de configuración, bases de datos, documentos de administración).

**Herramientas:**

- **`smbmap`**: Enumera y muestra permisos de SMB shares. 
- **`smbclient`**: Cliente SMB para interactuar con los shares.

- Los shares SMB suelen contener información sensible.
- Los archivos de configuración pueden contener contraseñas en texto plano.
- Pueden encontrarse scripts de automatización con credenciales embebidas.
- Los shares mal configurados (accesibles para "Todos" o "Domain Users") son un riesgo.


POLÍTICAS - `net accounts /domain`
de contraseñas del dominio:
    
    - Longitud mínima de contraseña.
        
    - Historial de contraseñas.
        
    - Duración máxima de contraseña.
        
    - Bloqueo de cuentas (intentos y duración).
        

**Herramienta:**

- **`net accounts /domain`**: Comando nativo de Windows.

**¿Por qué es importante?**

- Políticas débiles (ej: longitud mínima 7 caracteres) permiten ataques de fuerza bruta más fáciles.
- Si no hay bloqueo de cuentas, se puede hacer fuerza bruta sin ser detectado.
- Ayuda a entender qué tan **robustas** son las contraseñas del dominio y planificar ataques de diccionario.


ACLS PELIGROSAS - `BloodHound` / `PowerView`

**¿Qué se busca?**

- Identificar **ACLs (Access Control Lists)** mal configuradas:
    
    - Delegaciones de permisos excesivas.
        
    - Permisos de **escritura** en objetos críticos.
        
    - Capacidad de **modificar membresías de grupos**.
        
    - Permisos para **reseter contraseñas** de otros usuarios.
        
    - **GenericAll**, **GenericWrite**, **WriteOwner**, **WriteDACL** en objetos sensibles.
        

**Herramientas:**

- **`PowerView`**: Script de PowerShell para enumerar ACLs. 
- **`BloodHound`**: Herramienta gráfica que mapea relaciones y ACLs en el dominio, mostrando caminos de ataque visualmente.

**¿Por qué es importante?**

- Las ACLs mal configuradas son una de las principales vías de **escalada de privilegios**.
- Un usuario con permisos de escritura en un grupo de administradores puede agregarse a sí mismo.
- Permisos de reseteo de contraseña en un administrador pueden dar acceso total.
- BloodHound automatiza la búsqueda de estos caminos y los muestra gráficamente.

                    DOMAIN
                  corp.local
                       │
                       ▼
               ┌──────────────┐
               │    NMAP      │
               └──────┬───────┘
                      │
            Descubrimos servicios
                      │
        ┌─────────────┼──────────────┐
        ▼             ▼              ▼
       SMB           LDAP          Kerberos
        │             │              │
        ▼             ▼              ▼
     NetExec      ldapsearch     Impacket
     smbclient        │          ┌────┴─────┐
        │             │          ▼          ▼
        ▼             ▼    GetUserSPNs  GetNPUsers
     Shares        Usuarios      │           │
     Política      Grupos        ▼           ▼
                              SPN       DONT_PREAUTH
                                │           │
                                ▼           ▼
                         Kerberoasting  AS-REP