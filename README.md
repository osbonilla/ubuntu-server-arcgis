# Práctica: Servidor Ubuntu en VirtualBox + Conexión remota con MobaXterm

Guía paso a paso para levantar una máquina virtual con Ubuntu Server en VirtualBox y conectarse a ella de forma remota (SSH) desde MobaXterm en la máquina local.

---

## 1. Configuraciones de VirtualBox, link de descarga y versión de Ubuntu Server

### Descargas necesarias

| Software        | Versión recomendada                    | Enlace |
|------------------|-----------------------------------------|--------|
| VirtualBox       | Última estable (7.x)                    | https://www.virtualbox.org/wiki/Downloads |
| Ubuntu Server    | 24.04.3 LTS (Noble Numbat) — recomendada por estabilidad y soporte hasta 2029 | https://ubuntu.com/download/server |
| Ubuntu Server    | 26.04 LTS (Resolute Raccoon) — alternativa más reciente, soporte hasta 2031 | https://ubuntu.com/download/server |
| MobaXterm        | Home Edition (gratis)                   | https://mobaxterm.mobatek.net/download-home-edition.html |

> **Nota:** para esta práctica se recomienda usar una versión **LTS** (Long Term Support), ya que tiene más documentación disponible y es más estable para aprender.

### Configuración de la VM en VirtualBox

| Parámetro          | Valor recomendado                        |
|---------------------|-------------------------------------------|
| Tipo/Versión         | Linux / Ubuntu (64-bit)                   |
| Memoria RAM          | 2048 MB (mínimo 1024 MB)                  |
| Disco duro            | 20 GB, tipo VDI, reservado dinámicamente  |
| Procesadores          | 1-2 CPUs                                  |
| Red (Adaptador 1)     | **Adaptador puente (Bridged Adapter)**    |
| Unidad óptica          | ISO de Ubuntu Server montada como unidad de arranque |

**¿Por qué "Adaptador puente" y no NAT?**
Con Bridged, la VM obtiene su propia IP dentro de tu red local (como si fuera otro dispositivo físico), lo que facilita muchísimo la conexión SSH desde MobaXterm sin tener que configurar reenvío de puertos.

---

## 2. Instalación de Ubuntu Server

1. Crea la VM en VirtualBox con la configuración de la tabla anterior.
2. Inicia la VM y selecciona la ISO de Ubuntu Server descargada.
3. Sigue el instalador de texto (Subiquity):
   - Selecciona idioma y distribución de teclado.
   - Configura la red (normalmente detecta DHCP automáticamente).
   - Deja el "mirror" de paquetes por defecto.
   - En el particionado de disco, usa la opción **"Use an entire disk"** (más simple para prácticas).
   - Crea tu usuario, nombre de host y contraseña.
   - **Importante:** cuando el instalador pregunte por paquetes adicionales (SSH Setup), marca la opción **"Install OpenSSH server"**. Esto es clave para poder conectarte después desde MobaXterm.
   - No es necesario instalar snaps adicionales para esta práctica.
4. Espera a que termine la instalación, retira la ISO virtual y reinicia.
5. Inicia sesión con el usuario y contraseña creados.

---

## 3. Conexión entre máquina local y máquina virtual

### 3.1 Obtener la IP de la VM

Dentro de la VM (Ubuntu Server), ejecuta:

```bash
ip a
```

Busca la interfaz de red (por ejemplo `enp0s3`) y anota la IP que empieza normalmente con `192.168.x.x`.

### 3.2 Verificar que el servicio SSH está activo

```bash
sudo systemctl status ssh
```

Debe aparecer como `active (running)`. Si no está instalado o activo:

```bash
sudo apt update
sudo apt install openssh-server -y
sudo systemctl enable --now ssh
```

### 3.3 Permitir el puerto SSH en el firewall (si UFW está activo)

```bash
sudo ufw allow ssh
sudo ufw enable
sudo ufw status
```

### 3.4 Probar conectividad desde la máquina local

Desde una terminal de tu computadora local (CMD, PowerShell o terminal de MobaXterm):

```bash
ping <IP_DE_LA_VM>
```

Si responde, la red está bien configurada.

---

## 4. Instalación de MobaXterm

1. Descarga la versión **Home Edition (Installer edition)** desde el link oficial.
2. Ejecuta el instalador y sigue los pasos por defecto.
3. Abre MobaXterm.
4. Ve a **Session → New session → SSH**.
5. Completa los campos:
   - **Remote host:** IP de la VM (obtenida en el paso 3.1)
   - **Specify username:** el usuario creado durante la instalación de Ubuntu
   - **Port:** 22
6. Haz clic en **OK**.
7. Acepta la advertencia de host key (primera conexión) y escribe la contraseña del usuario.

---

## 5. Comprobación

Una vez conectado desde MobaXterm, verifica que estás dentro de la VM:

```bash
whoami
hostname
uname -a
```

Deberías ver el nombre de usuario y el hostname que definiste durante la instalación de Ubuntu Server, confirmando que estás controlando la VM de forma remota y no tu máquina local.

---

## 6. Comandos de prueba

Comandos básicos para validar que todo funciona correctamente vía SSH:

```bash
# Información del sistema
uname -a                # Info del kernel y arquitectura
lsb_release -a           # Versión de Ubuntu instalada

# Navegación
pwd                      # Directorio actual
ls -la                   # Listar archivos (incluyendo ocultos)
cd /var/log              # Cambiar de directorio

# Gestión de archivos
touch prueba.txt         # Crear archivo
echo "hola mundo" > prueba.txt   # Escribir contenido
cat prueba.txt           # Ver contenido
rm prueba.txt            # Eliminar archivo

# Info de red
ip a                     # Ver IP y adaptadores
hostname -I               # IP rápida

# Info de sistema y recursos
top                       # Procesos en tiempo real (salir con 'q')
df -h                     # Espacio en disco
free -h                   # Uso de memoria RAM

# Gestión de paquetes
sudo apt update           # Actualizar índices de paquetes
sudo apt list --upgradable # Ver paquetes actualizables
```

Si todos estos comandos se ejecutan sin errores, la práctica se considera exitosa: tienes una VM con Ubuntu Server funcionando y accesible de forma remota vía SSH desde MobaXterm.

---

## Resumen del flujo completo

```
[Tu PC] --VirtualBox--> [VM Ubuntu Server] --Bridged (misma red)--> IP propia
   |
   └── MobaXterm (SSH, puerto 22) ---> Conexión remota a la VM
```

---

## Problemas comunes

| Problema | Posible causa | Solución |
|----------|----------------|----------|
| No responde el `ping` | Modo de red incorrecto | Verifica que sea "Bridged Adapter" y no NAT |
| MobaXterm no conecta | SSH no instalado/activo | `sudo systemctl enable --now ssh` |
| Conexión rechazada | Firewall bloqueando puerto 22 | `sudo ufw allow ssh` |
| IP cambia cada reinicio | DHCP dinámico | Considera fijar IP estática en netplan (opcional, para prácticas avanzadas) |