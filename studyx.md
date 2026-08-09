┌──(kode㉿localhost)-[~/Downloads/rustscan.deb]  
└─$ nmap --script mysql-empty-password,mysql-databases 209.126.84.123  
Starting Nmap 7.99 ( https://nmap.org ) at 2026-04-30 14:32 -0400  
Nmap scan report for vmi580484.contaboserver.net (209.126.84.123)  
Host is up (0.027s latency).  
Not shown: 313 filtered tcp ports (admin-prohibited), 199 filtered tcp ports (no-response), 4  
72 closed tcp ports (reset)  
PORT     STATE SERVICE  
21/tcp   open  ftp  
22/tcp   open  ssh  
25/tcp   open  smtp  
80/tcp   open  http  
110/tcp  open  pop3  
111/tcp  open  rpcbind  
143/tcp  open  imap  
443/tcp  open  https  
465/tcp  open  smtps  
554/tcp  open  rtsp  
587/tcp  open  submission  
993/tcp  open  imaps  
995/tcp  open  pop3s  
1723/tcp open  pptp  
3000/tcp open  ppp  
3306/tcp open  mysql