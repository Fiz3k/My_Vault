### Guía de arranque del laboratorio

---

### Lab Active Directory — solo AD

#### En el T14

bash

```bash
# Arrancar VMs
vboxmanage startvm "DC01" --type headless
vboxmanage startvm "WS01" --type headless
vboxmanage startvm "WS02" --type headless
```

bash

```bash
# Verificar bridge
brctl show | grep eth0
# Debe mostrar eth0 bajo br0
```

bash

```bash
# Si br0 no tiene eth0
sudo ip link set eth0 master br0
sudo ip link set br0 up
```

bash

```bash
# Esperar 60 segundos y verificar VMs
sleep 60 && nmap -sn 192.168.10.100-102
```

#### En el X1 Carbon

bash

```bash
# Verificar IP
ip addr show eth0 | grep 192.168.10.20

# Si no tiene IP
sudo nmcli connection up "lab-eth0"
```

bash

```bash
# Verificar lab AD
crackmapexec smb 192.168.10.100-102
```

✅ Lab AD listo cuando aparezcan DC01, WS01, WS02.

---

### Lab Web — solo aplicaciones web

#### En el T14

bash

```bash
# Verificar bridge
brctl show | grep eth0
sudo ip link set eth0 master br0 2>/dev/null
sudo ip link set br0 up
```

bash

```bash
# Arrancar contenedores Docker
docker start dvwa bwapp juiceshop webgoat
```

bash

```bash
# Arrancar NodeGoat
cd ~/lab-ad/docker/nodegoat
docker-compose up -d
```

bash

```bash
# Verificar todos corriendo
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```

#### En el X1 Carbon

bash

```bash
# Verificar IP
ip addr show eth0 | grep 192.168.10.20
sudo nmcli connection up "lab-eth0" 2>/dev/null
```

bash

```bash
# Verificar todos los targets
curl -s -o /dev/null -w "DVWA:      %{http_code}\n" http://192.168.10.10
curl -s -o /dev/null -w "bWAPP:     %{http_code}\n" http://192.168.10.10:8080
curl -s -o /dev/null -w "JuiceShop: %{http_code}\n" http://192.168.10.10:3000
curl -s -o /dev/null -w "WebGoat:   %{http_code}\n" http://192.168.10.10:8888/WebGoat
curl -s -o /dev/null -w "NodeGoat:  %{http_code}\n" http://192.168.10.10:4000
```

✅ Lab Web listo cuando todos retornen 200 o 302.

---

### Lab completo — AD + Web simultáneo

#### En el T14

bash

```bash
# Arrancar todo
vboxmanage startvm "DC01" --type headless
vboxmanage startvm "WS01" --type headless
vboxmanage startvm "WS02" --type headless
vboxmanage startvm "Metasploitable 2" --type headless
vboxmanage startvm "Metasploitable3-ub1404" --type headless
vboxmanage startvm "metasploitable3 DCSE2018v2" --type headless
docker start dvwa bwapp juiceshop webgoat
cd ~/lab-ad/docker/nodegoat && docker-compose up -d
```

bash

```bash
# Verificar bridge
brctl show | grep eth0
sudo ip link set eth0 master br0 2>/dev/null
```

bash

```bash
# Verificar todo el lab
sleep 60 && nmap -sn 192.168.10.0/24
```

#### En el X1 Carbon

bash

```bash
sudo nmcli connection up "lab-eth0" 2>/dev/null
nmap -sn 192.168.10.0/24
```

---

### Scripts de arranque rápido

#### En el T14 — guardar script

bash

```bash
cat > ~/lab-start.sh << 'EOF'
#!/bin/bash
echo "[*] Verificando bridge..."
sudo ip link set eth0 master br0 2>/dev/null
sudo ip link set br0 up

if [ "$1" = "ad" ] || [ "$1" = "all" ]; then
    echo "[*] Arrancando lab AD..."
    vboxmanage startvm "DC01" --type headless
    vboxmanage startvm "WS01" --type headless
    vboxmanage startvm "WS02" --type headless
fi

if [ "$1" = "web" ] || [ "$1" = "all" ]; then
    echo "[*] Arrancando lab Web..."
    docker start dvwa bwapp juiceshop webgoat
    cd ~/lab-ad/docker/nodegoat && docker-compose up -d
fi

echo "[*] Esperando 60 segundos..."
sleep 60
echo "[*] Estado del lab:"
nmap -sn 192.168.10.0/24
EOF
chmod +x ~/lab-start.sh
```

Uso:

bash

```bash
~/lab-start.sh ad     # Solo Active Directory
~/lab-start.sh web    # Solo Web
~/lab-start.sh all    # Todo el laboratorio
```

#### En el X1 Carbon — guardar script

bash

```bash
cat > ~/lab-connect.sh << 'EOF'
#!/bin/bash
echo "[*] Conectando al lab..."
sudo nmcli connection up "lab-eth0" 2>/dev/null
sleep 3

echo "[*] Verificando AD..."
ping -c 1 -W 1 192.168.10.100 > /dev/null && echo "DC01: UP" || echo "DC01: DOWN"
ping -c 1 -W 1 192.168.10.101 > /dev/null && echo "WS01: UP" || echo "WS01: DOWN"
ping -c 1 -W 1 192.168.10.102 > /dev/null && echo "WS02: UP" || echo "WS02: DOWN"

echo "[*] Verificando Web..."
curl -s -o /dev/null -w "DVWA:      %{http_code}\n" http://192.168.10.10
curl -s -o /dev/null -w "bWAPP:     %{http_code}\n" http://192.168.10.10:8080
curl -s -o /dev/null -w "JuiceShop: %{http_code}\n" http://192.168.10.10:3000
curl -s -o /dev/null -w "WebGoat:   %{http_code}\n" http://192.168.10.10:8888/WebGoat
curl -s -o /dev/null -w "NodeGoat:  %{http_code}\n" http://192.168.10.10:4000
EOF
chmod +x ~/lab-connect.sh
```

Uso:

bash

```bash
~/lab-connect.sh
```