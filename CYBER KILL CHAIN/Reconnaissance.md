Reconocimiento Pasivo
- OSINT
- Google Dorks

Reconocimiento Activo
- Nmap `nmap -sV -sC -p- -T4 <dirección_IP>`

| **Categoria**                        | **Comando / Extensión**         | **Descripción**                                                                 |
| ------------------------------------ | ------------------------------- | ------------------------------------------------------------------------------- |
| **Descubrimiento de Hosts**          | `-sn`                           | Escaneo Ping (desactiva el escaneo de puertos).                                 |
|                                      | `-Pn`                           | Omite el descubrimiento de hosts (asume que el objetivo está activo).           |
|                                      | `-PR`                           | Descubrimiento mediante ARP Ping (muy rápido en redes locales).                 |
| **Técnicas de Escaneo**              | `-sS`                           | Escaneo SYN (_TCP Half-Open_), rápido y discreto (requiere root/sudo).          |
|                                      | `-sT`                           | Escaneo TCP Connect (completa el saludo de 3 vías, usado sin privilegios).      |
|                                      | `-sU`                           | Escaneo de puertos UDP.                                                         |
|                                      | `-sA`                           | Escaneo ACK (usado para mapear reglas de cortafuegos).                          |
| **Especificación de Puertos**        | `-p <rango>`                    | Escanea puertos específicos (ej. `-p 80,443` o `-p 1-1024`).                    |
|                                      | `-p-`                           | Escanea la totalidad de los 65,535 puertos TCP.                                 |
|                                      | `-F`                            | Escaneo rápido (_Fast scan_), analiza los 100 puertos más comunes.              |
| **Detección de Servicios y S.O.**    | `-sV`                           | Determina la versión del servicio que corre en los puertos abiertos.            |
|                                      | `-O`                            | Activa la detección del Sistema Operativo del objetivo.                         |
|                                      | `-A`                            | Modo agresivo (activa `-O`, `-sV`, escaneo de scripts `-sC` y `traceroute`).    |
| **Motor de Scripts (NSE)**           | `-sC`                           | Ejecuta los scripts por defecto (_default set_).                                |
|                                      | `--script=<script>`             | Ejecuta un script o categoría específica (ej. `--script=vuln`).                 |
| **Rendimiento y Tiempo**             | `-T0` a `-T5`                   | Plantillas de tiempo: desde Paranoid (`-T0`, ultra lento) hasta Insane (`-T5`). |
|                                      | `--max-rtt-timeout`             | Define el tiempo máximo de espera por respuesta (_round trip time_).            |
| **Evasión de Seguridad / Firewalls** | `-f`                            | Fragmenta los paquetes para dificultar la detección por IDS/Firewalls.          |
|                                      | `-D <decoy1,decoy2>`            | Utiliza señuelos (_decoys_) para ocultar la IP origen real.                     |
|                                      | `-S <IP>`                       | Suplanta la dirección IP de origen (_IP Spoofing_).                             |
|                                      | `-g <puerto>` / `--source-port` | Utiliza un puerto de origen específico (ej. 53 o 80).                           |
| **Salida de Resultados**             | `-oN <archivo>`                 | Guarda la salida en formato de texto normal.                                    |
|                                      | `-oX <archivo>`                 | Guarda la salida en formato XML.                                                |
|                                      | `-oG <archivo>`                 | Guarda la salida en formato interpretable por Grep.                             |
|                                      | `-oA <nombre>`                  | Guarda el resultado en los tres formatos principales a la vez.                  |
