# Guía práctica de comandos Linux (con redes, servicios e instalación)

> **Formato:** Markdown (.md) • **Enfoque:** explicaciones paso a paso con ejemplos prácticos • **Distros objetivo:** Debian/Ubuntu, RHEL/CentOS/Rocky, Arch, openSUSE (se indican variantes)

---

## 0) Fundamentos y nota rápida sobre distros

- **Usuario super:** `root` o `sudo` antes de un comando.  
- **Gestores de paquetes (según distro):**
  - Debian/Ubuntu: `apt` (`apt-get` legado)
  - RHEL/CentOS/Rocky/Alma: `dnf` (o `yum` legado)
  - Arch: `pacman`
  - openSUSE: `zypper`
  - Universales: `snap`, `flatpak`
- **Servicios:** `systemd` (comando `systemctl`) en la mayoría de distros actuales.
- **Archivos importantes:** `/etc/` (config), `/var/log/` (logs), `/etc/systemd/system/` (servicios propios).

---

## 1) Comandos básicos del sistema

### 1.1 Información del sistema
- `uname -a` — kernel y arquitectura.
- `lsb_release -a` (Deb/Ubuntu) — versión de la distro.
- `cat /etc/os-release` — info universal de la distribución.
- `hostnamectl` — ver/cambiar hostname.
- `timedatectl` — zona horaria y sincronización NTP.

### 1.2 Navegación y archivos
- `pwd` — ruta actual.
- `ls -lah` — listar con detalles y tamaños legibles.
- `cd /ruta` — cambiar de carpeta. `cd -` vuelve a la anterior.
- `mkdir -p dir/subdir` — crear carpetas recursivamente.
- `touch archivo` — crear/modificar marca de tiempo.
- `cp -r origen destino` — copiar. `mv` mueve/renombra. `rm -rf` elimina (⚠️ cuidado).
- `ln -s origen enlace` — enlace simbólico.

### 1.3 Ver y buscar dentro de archivos
- `cat`, `less`, `head`, `tail -f /var/log/syslog` — lectura (seguimiento en vivo con `-f`).
- `grep -R "texto" /ruta` — buscar texto recursivo. Útil: `-n` (número de línea), `-i` (insensible a mayúsculas).
- `find /ruta -name "*.log" -size +100M` — buscar por nombre/tamaño.
- `awk`, `sed` — procesado de texto (ver ejemplos en sección 3).

### 1.4 Compresión y empaquetado
- `tar -czvf backup.tgz /carpeta` — comprimir.
- `tar -xzvf backup.tgz -C /destino` — descomprimir.
- `zip -r archivo.zip carpeta` / `unzip archivo.zip` — zip.

### 1.5 Disco y uso
- `df -h` — uso de discos.
- `du -sh * | sort -h` — tamaño de carpetas/archivos en directorio actual.
- `mount`, `lsblk`, `blkid` — dispositivos y montajes.

---

## 2) Administración de usuarios y permisos

### 2.1 Usuarios y grupos
- Crear usuario:  
  - Debian/Ubuntu: `sudo adduser juan` (interactivo)  
  - Genérico: `sudo useradd -m juan && sudo passwd juan`
- Añadir a grupo sudo/admin:  
  - Debian/Ubuntu: `sudo usermod -aG sudo juan`  
  - RHEL: `sudo usermod -aG wheel juan`
- Listar grupos de un usuario: `groups juan`

### 2.2 Permisos, propietarios y umask
- `chown usuario:grupo archivo` — cambiar propietario.
- `chmod 644 archivo` / `chmod 755 carpeta` — permisos (r=4, w=2, x=1).
- `umask 022` — máscara por defecto (permiso *restado* a nuevos archivos).

### 2.3 sudoers
- Editar de forma segura: `sudo visudo`
- Dar permisos sin contraseña a un comando específico (ejemplo):  
  ```
  juan ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart mi-servicio
  ```

---

## 3) Archivos y procesamiento de texto (power moves)

- Contar líneas/palabras: `wc -l`, `wc -w archivo`
- Extraer columnas: `awk '{print $1,$3}' archivo.txt`
- Reemplazo in-place: `sed -i 's/viejo/nuevo/g' archivo`
- Filtrar y ordenar: `cat acces.log | grep "200" | awk '{print $1}' | sort | uniq -c | sort -nr | head`

---

## 4) Procesos y servicios

### 4.1 Procesos
- `ps aux | grep nombre` — listar procesos.
- `top` o `htop` — monitoreo interactivo (instalar `htop` si no está).
- `pgrep nginx` / `pkill -f python`
- Prioridad: `nice -n 10 comando`, `renice 5 -p <PID>`
- Señales: `kill -TERM <PID>`, `kill -9 <PID>` (forzado).

### 4.2 Jobs en segundo plano / sesiones
- `nohup comando &` — persistir si cierras sesión.
- `screen` o `tmux` — multiplexores de terminal (recomendado para servidores).

### 4.3 Cron (tareas programadas)
- Editar crontab del usuario: `crontab -e`
- Ejemplo: ejecutar cada día a las 03:30:  
  ```
  30 3 * * * /usr/local/bin/backup.sh >> /var/log/backup.log 2>&1
  ```

---

## 5) Red y diagnóstico

### 5.1 Estado de red e interfaces
- `ip a` — interfaces y direcciones.
- `ip r` — rutas.
- `ping 8.8.8.8` / `ping -c 4 dominio.com`
- `ss -tulpen` — sockets escuchando (TCP/UDP) con procesos. Reemplaza a `netstat`.
- `traceroute dominio.com` o `mtr dominio.com` — traza de ruta (instalar si falta).
- DNS: `dig dominio.com +short`, `nslookup dominio.com` (legacy).

### 5.2 Pruebas HTTP/HTTPS y transferencia
- `curl -I https://ejemplo.com` — solo cabeceras.
- `curl -v https://api.ejemplo.com/health`
- `wget URL` — descarga.
- `scp archivo usuario@host:/ruta` — copia segura entre hosts.
- `rsync -avz origen/ usuario@host:/destino/` — sincronización eficiente.

### 5.3 Configurar IP estática con `nmcli` (NetworkManager)
> Útil en distros con NetworkManager (Ubuntu Server reciente, RHEL/Rocky, Fedora, etc.).

1) Listar conexiones:  
   `nmcli con show`
2) Establecer IPv4 manual:  
   ```bash
   sudo nmcli con mod "enp0s3" ipv4.addresses "192.168.1.50/24" \
     ipv4.gateway "192.168.1.1" ipv4.dns "1.1.1.1,8.8.8.8" ipv4.method manual
   ```
3) Subir la conexión:  
   `sudo nmcli con up "enp0s3"`

### 5.4 Firewall básico
- **UFW (Ubuntu):**
  ```bash
  sudo apt install ufw
  sudo ufw default deny incoming
  sudo ufw default allow outgoing
  sudo ufw allow 22/tcp    # SSH
  sudo ufw allow 80,443/tcp
  sudo ufw enable
  sudo ufw status verbose
  ```
- **firewalld (RHEL/Fedora/Rocky):**
  ```bash
  sudo dnf install firewalld
  sudo systemctl enable --now firewalld
  sudo firewall-cmd --add-service=http --permanent
  sudo firewall-cmd --add-service=https --permanent
  sudo firewall-cmd --reload
  sudo firewall-cmd --list-all
  ```

### 5.5 Captura de paquetes con `tcpdump`
```bash
sudo tcpdump -i eth0 -nn port 80
sudo tcpdump -i eth0 -nn host 192.168.1.10
sudo tcpdump -i eth0 -w captura.pcap  # abrir con Wireshark
```

---

## 6) Instalación y gestión de paquetes

### 6.1 Comandos por familia

- **Debian/Ubuntu (APT):**
  ```bash
  sudo apt update
  sudo apt install paquete
  sudo apt upgrade
  sudo apt remove paquete
  sudo apt autoremove
  apt show paquete
  apt list --installed | grep nginx
  ```

- **RHEL/CentOS/Rocky (DNF/YUM):**
  ```bash
  sudo dnf makecache
  sudo dnf install paquete
  sudo dnf upgrade
  sudo dnf remove paquete
  dnf info paquete
  ```

- **Arch (pacman):**
  ```bash
  sudo pacman -Syu
  sudo pacman -S paquete
  sudo pacman -Rns paquete
  pacman -Qi paquete
  ```

- **openSUSE (zypper):**
  ```bash
  sudo zypper refresh
  sudo zypper install paquete
  sudo zypper update
  sudo zypper remove paquete
  zypper info paquete
  ```

- **Universales:**
  ```bash
  # Snap
  sudo snap install nombre
  snap list

  # Flatpak
  flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
  flatpak install flathub org.example.App
  flatpak list
  ```

### 6.2 Ejemplo: instalar NGINX y habilitarlo
```bash
# Debian/Ubuntu
sudo apt update && sudo apt install -y nginx
sudo systemctl enable --now nginx
sudo systemctl status nginx
curl -I http://localhost

# RHEL/Rocky
sudo dnf install -y nginx
sudo systemctl enable --now nginx
```

---

## 7) Creación y gestión de servicios (systemd)

### 7.1 Comandos esenciales de `systemctl`
- Estado y control:
  ```bash
  sudo systemctl status nombre.service
  sudo systemctl start nombre.service
  sudo systemctl stop nombre.service
  sudo systemctl restart nombre.service
  sudo systemctl reload nombre.service
  sudo systemctl enable nombre.service   # iniciar al arrancar
  sudo systemctl disable nombre.service
  sudo systemctl is-enabled nombre.service
  ```

- Logs con `journalctl`:
  ```bash
  sudo journalctl -u nombre.service -f    # seguir en vivo
  sudo journalctl -u nombre.service --since "1 hour ago"
  sudo journalctl -b                      # logs del último arranque
  ```

### 7.2 Crear un servicio para una app (ejemplo Node.js)
**Supuestos:** app en `/opt/miapp` ejecuta `npm start` y usa puerto 3000.

1) Crear usuario de servicio (opcional pero recomendado):
```bash
sudo useradd -r -s /bin/false miapp
sudo mkdir -p /opt/miapp && sudo chown -R miapp:miapp /opt/miapp
```

2) Crear unidad systemd `/etc/systemd/system/miapp.service`:
```ini
[Unit]
Description=Mi App Node
After=network.target

[Service]
User=miapp
WorkingDirectory=/opt/miapp
ExecStart=/usr/bin/npm start
Restart=on-failure
Environment=NODE_ENV=production
# Asegura PATH si usas nvm/pm2/etc:
# Environment=PATH=/usr/local/bin:/usr/bin

[Install]
WantedBy=multi-user.target
```

3) Recargar y habilitar:
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now miapp.service
sudo systemctl status miapp.service
sudo journalctl -u miapp.service -f
```

### 7.3 Servicio *templated* (múltiples instancias)
Archivo: `/etc/systemd/system/miworker@.service`
```ini
[Unit]
Description=Worker %i
After=network.target

[Service]
User=miapp
ExecStart=/usr/local/bin/worker --queue %i
Restart=always

[Install]
WantedBy=multi-user.target
```
Activar dos instancias:
```bash
sudo systemctl enable --now miworker@email.service
sudo systemctl enable --now miworker@sms.service
```

---

## 8) Seguridad y monitoreo

### 8.1 Usuarios, SSH y sudo
- Deshabilitar root por SSH: editar `/etc/ssh/sshd_config` → `PermitRootLogin no` y reiniciar `sudo systemctl restart sshd`.
- Autenticación por claves: generar con `ssh-keygen`, copiar con `ssh-copy-id usuario@host`.
- `sudo` granular (ver sección 2.3).

### 8.2 Firewall y endurecimiento básico
- Configura **UFW** o **firewalld** (sección 5.4).
- Servicios expuestos: valida con `ss -tulpen`.
- Actualizaciones regulares: `sudo apt upgrade` / `sudo dnf upgrade`.
- Permisos correctos en claves: `chmod 600 ~/.ssh/id_rsa`

### 8.3 Monitoreo del sistema
- Memoria/CPU:
  - `free -h`, `vmstat 1 5`, `top/htop`, `uptime`
- IO/Disco:
  - `iostat -xz 1` (paquete `sysstat`), `iotop`
- Redes:
  - `nload`, `iftop`, `ss`, `sar -n DEV 1 5`
- Logs:
  - `journalctl`, `/var/log/*`

### 8.4 Fail2ban (bloqueo por intentos fallidos)
```bash
sudo apt install fail2ban  # o sudo dnf install fail2ban
sudo systemctl enable --now fail2ban
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local  # activar [sshd] y ajustar bantime/findtime/maxretry
sudo systemctl restart fail2ban
sudo fail2ban-client status sshd
```

---

## 9) Flujos paso a paso (mini-recetas)

### 9.1 Diagnóstico rápido de “no carga mi web”
1. ¿El servicio corre? `systemctl status nginx` / `journalctl -u nginx -f`
2. ¿Escucha el puerto? `ss -tulpen | grep :80`
3. ¿Firewall abierto? `ufw status` o `firewall-cmd --list-all`
4. ¿Conectividad? `curl -I http://127.0.0.1` y desde otro host `curl -I http://IP`
5. ¿DNS apunta? `dig midominio.com +short`
6. Logs del sitio: `/var/log/nginx/access.log` y `error.log`

### 9.2 Habilitar HTTPS con certbot (NGINX en Ubuntu)
```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d midominio.com -d www.midominio.com
sudo systemctl status nginx
```

### 9.3 Crear un servicio systemd para un script Python
1) Script en `/opt/backup/backup.py` que haga algo útil.  
2) Unidad `/etc/systemd/system/backup.service`:
```ini
[Unit]
Description=Backup diario

[Service]
Type=oneshot
ExecStart=/usr/bin/python3 /opt/backup/backup.py
```
3) **Timer** `/etc/systemd/system/backup.timer`:
```ini
[Unit]
Description=Ejecutar backup diario

[Timer]
OnCalendar=02:30
Persistent=true

[Install]
WantedBy=timers.target
```
4) Activar:
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now backup.timer
systemctl list-timers | grep backup
```

### 9.4 Montar disco y persistir en `/etc/fstab`
```bash
lsblk                           # identificar /dev/sdb1
sudo mkdir -p /mnt/datos
sudo blkid /dev/sdb1            # copiar UUID="..."
# Editar /etc/fstab y añadir línea, por ejemplo:
UUID=xxxx-xxxx /mnt/datos ext4 defaults,noatime 0 2
sudo mount -a
```

---

## 10) Referencias rápidas (cheat sheet compacto)

- **Sistema:** `uname -a`, `hostnamectl`, `timedatectl`, `top`, `htop`
- **Archivos:** `ls`, `cd`, `cp`, `mv`, `rm`, `ln -s`, `grep`, `find`, `awk`, `sed`, `tar`, `zip/unzip`
- **Usuarios:** `adduser/useradd`, `usermod -aG`, `passwd`, `chown`, `chmod`, `visudo`
- **Paquetes:** `apt/dnf/pacman/zypper`, `snap`, `flatpak`
- **Red:** `ip a`, `ip r`, `ss -tulpen`, `ping`, `traceroute/mtr`, `dig`, `curl`, `wget`, `scp`, `rsync`
- **Servicios:** `systemctl [status|start|enable]`, `journalctl -u`, `ufw`/`firewalld`
- **Monitoreo:** `free -h`, `vmstat`, `iostat`, `iotop`, `sar`, `journalctl`

---

## 11) Buenas prácticas
- Automatiza con **scripts** (bash/python) y **systemd timers**.
- Mantén **backups** y **restores** probados.
- Documenta cambios en `/etc/` (usa git para `/etc` si aplica).
- Sé cuidadoso con `sudo rm -rf` y valida rutas dos veces.
- Actualiza con regularidad y revisa los **logs** tras cambios.

---

¿Quieres que la convierta también a PDF o que genere una versión “solo comandos” tipo chuleta para imprimir?
