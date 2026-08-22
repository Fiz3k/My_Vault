**192.168.10.1** → T14 (vboxnet0) 192.168.10.10 → T14 (eth0 física) **192.168.10.20** → X1 Carbon (atacante) 
**192.168.10.100** → DC01 (Domain Controller) 
**192.168.10.101** → WS01 (Workstation víctima 1) 
**192.168.10.102** → WS02 (Workstation víctima 2)

### Estado del laboratorio

#### Active Directory

|IP|Servicio|Estado|
|---|---|---|
|192.168.10.100|DC01 — Domain Controller|✅|
|192.168.10.101|WS01 — Workstation|✅|
|192.168.10.102|WS02 — Workstation|✅|

#### Servicios vulnerables de red

|IP|Servicio|Estado|
|---|---|---|
|192.168.10.110|Metasploitable2|✅|
|192.168.10.111|Metasploitable3 Ubuntu|✅|
|192.168.10.112|Metasploitable3 Windows|✅|

#### Aplicaciones web vulnerables

|URL|App|Credenciales|
|---|---|---|
|[http://192.168.10.10](http://192.168.10.10)|DVWA|admin / password|
|[http://192.168.10.10:8080](http://192.168.10.10:8080)|bWAPP|bee / bug|
|[http://192.168.10.10:3000](http://192.168.10.10:3000)|Juice Shop|registro libre|
|[http://192.168.10.10:8888/WebGoat](http://192.168.10.10:8888/WebGoat)|WebGoat|registro libre|
|[http://192.168.10.10:4000](http://192.168.10.10:4000)|NodeGoat|admin / Admin_123|

---

### Ataques disponibles

#### Active Directory

|Ataque|Herramienta|Target|
|---|---|---|
|AS-REP Roasting ✅|impacket GetNPUsers|jsmith|
|Kerberoasting|impacket GetUserSPNs|svc_sql, svc_iis|
|Password Spray|CrackMapExec|todos|
|LLMNR Poisoning|Responder|red completa|
|BloodHound Enum|bloodhound-python|AD completo|
|DCSync|secretsdump|bjohnson|

#### Web

| Ataque             | App recomendada |
| ------------------ | --------------- |
| SQLi               | DVWA, WebGoat   |
| XSS                | DVWA, bWAPP     |
| CSRF               | bWAPP, WebGoat  |
| XXE                | bWAPP           |
| IDOR / Auth bypass | Juice Shop      |
| JWT attacks        | NodeGoat        |
| LFI/RFI            | DVWA, bWAPP     |