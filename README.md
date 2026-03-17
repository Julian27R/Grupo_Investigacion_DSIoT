# 🖥️ Servidor IoT Local con Ubuntu, Docker y ThingsBoard

Proyecto de aprendizaje para montar un servidor local completo con capacidades IoT, usando un mini-PC, contenedores Docker y la plataforma ThingsBoard.

---

## 📌 ¿Qué hace este proyecto?

Este proyecto te guía paso a paso para convertir un mini-PC con Ubuntu en un servidor local IoT capaz de:

- Recibir datos de sensores (reales o simulados)
- Visualizarlos en dashboards en tiempo real
- Ser accesible desde cualquier dispositivo en la misma red

No necesitas experiencia previa — cada sección explica el concepto antes de ejecutar comandos.

---

## 🧠 Conceptos clave

**¿Qué es un servidor?**
Un servidor es un computador que ofrece servicios a otros dispositivos en la red. En este proyecto, tu mini-PC será el servidor.

**¿Qué es Docker?**
Docker permite "empaquetar" programas en contenedores — como cajas selladas que incluyen todo lo necesario para que una aplicación funcione, sin afectar el resto del sistema.

**¿Qué es IoT?**
IoT (Internet of Things) es la red de dispositivos físicos (sensores, microcontroladores) que envían datos a través de internet o redes locales.

**¿Qué es ThingsBoard?**
ThingsBoard es una plataforma de código abierto para recibir, gestionar y visualizar datos IoT desde dispositivos conectados.

---

## 🧱 Arquitectura del sistema

```
[Simulador Python o Dispositivo Real]
              │
              │  HTTP o MQTT
              ▼
     ThingsBoard (Docker, puerto 8080)
              │
              ▼
       Dashboard Web (navegador)

Mini-PC (Servidor)
├── Puerto 8080 → ThingsBoard (IoT)
└── Puerto 8090 → NGINX (Web)
```

---

## ⚙️ Tecnologías utilizadas

| Tecnología | Función |
|---|---|
| Ubuntu 22.04 LTS | Sistema operativo del servidor |
| Docker + Docker Compose | Gestión de contenedores |
| ThingsBoard (`tb-postgres`) | Plataforma IoT con base de datos integrada |
| NGINX | Servidor web |
| Python 3 | Simulador de dispositivos IoT |
| SSH / VS Code Remote | Acceso remoto al servidor |

---

## 🗂️ Estructura de carpetas

```
~/server/
├── nginx/
│   └── docker-compose.yml
└── thingsboard/
    └── docker-compose.yml
```

---

## 🖥️ Paso 1 — Preparar Ubuntu

Ubuntu 22.04 LTS es la versión recomendada para servidores por su estabilidad y 5 años de soporte oficial.

### Actualizar el sistema

```bash
sudo apt update
sudo apt upgrade -y
```

### Identificar la IP del servidor

```bash
ip a
```

Busca una línea como: `inet 172.20.25.173/21`

> 📌 Anota esta IP — la usarás en todos los pasos siguientes.

---

## 🔐 Paso 2 — Acceso remoto con SSH

```bash
# Instalar SSH
sudo apt install openssh-server -y

# Verificar que está activo
sudo systemctl status ssh

# Conectarse desde otro equipo
ssh tu_usuario@IP_DEL_SERVIDOR
```

---

## 💻 Paso 3 (opcional) — Conectar VS Code al servidor

1. Instala la extensión **Remote - SSH** en VS Code
2. Edita `~/.ssh/config`:

```
Host servidor-iot
    HostName 172.20.25.173
    User pci
```

3. En VS Code: `Ctrl + Shift + P` → `Remote-SSH: Connect to Host` → selecciona `servidor-iot`

---

## 🐳 Paso 4 — Instalar Docker y Docker Compose

```bash
# Verificar si ya está instalado
docker --version
docker compose version

# Instalar Docker Compose si no está disponible
sudo apt install docker-compose-plugin -y

# Probar que Docker funciona
docker run hello-world
```

---

## 📁 Paso 5 — Crear la estructura de carpetas

```bash
mkdir -p ~/server/nginx
mkdir -p ~/server/thingsboard
```

---

## 🌍 Paso 6 — Levantar NGINX (servidor web)

```bash
nano ~/server/nginx/docker-compose.yml
```

Contenido:

```yaml
services:
  nginx:
    image: nginx:latest
    container_name: nginx_web_server
    ports:
      - "8090:80"
    restart: unless-stopped
```

```bash
cd ~/server/nginx
docker compose up -d
```

---

## 📡 Paso 7 — Desplegar ThingsBoard (plataforma IoT)

> **Nota:** Se usa la imagen `thingsboard/tb-postgres` en lugar de `tb-edge`.  
> Esta imagen incluye base de datos integrada (PostgreSQL), arranca sin configuración adicional y es la opción recomendada para servidores locales.  
> `tb-edge` está diseñada para conectarse a una instancia cloud de ThingsBoard, lo que requiere configuración extra.

### 7.1 — Crear el docker-compose.yml

```bash
nano ~/server/thingsboard/docker-compose.yml
```

Contenido:

```yaml
services:
  thingsboard:
    image: thingsboard/tb-postgres
    container_name: thingsboard
    ports:
      - "8080:9090"          # Web UI
      - "1883:1883"          # MQTT
      - "7070:7070"          # Edge RPC
      - "5683-5688:5683-5688/udp"  # CoAP
    environment:
      TB_QUEUE_TYPE: in-memory
    volumes:
      - tb-data:/data
      - tb-logs:/var/log/thingsboard
    restart: unless-stopped

volumes:
  tb-data:
  tb-logs:
```

### 7.2 — Levantar el servicio

```bash
cd ~/server/thingsboard
docker compose up -d

# Seguir los logs hasta que esté listo (~60-90 segundos)
docker compose logs -f
```

Espera hasta ver: `Started ThingsboardServerApplication in XX seconds`  
Presiona `Ctrl + C` para salir de los logs (el contenedor sigue corriendo).

### 7.3 — Acceder desde el navegador

```
http://IP_DEL_SERVIDOR:8080
```

**Credenciales por defecto:**

| Campo | Valor |
|---|---|
| Usuario | `tenant@thingsboard.org` |
| Contraseña | `tenant` |

> ⚠️ Cambia la contraseña antes de usar en producción.

---

## 🔑 Paso 8 — Crear un dispositivo IoT en ThingsBoard

1. Entra a ThingsBoard → sección **Devices**
2. Haz clic en **Add Device**
3. Asigna un nombre, por ejemplo: `Sensor_Prueba_1`
4. Entra al dispositivo → pestaña **Credentials**
5. Copia el **Access Token** — lo necesitarás en el simulador

---

## 🧪 Paso 9 — Simular un dispositivo IoT

### Instalar dependencias

```bash
pip3 install requests paho-mqtt
```

### Opción A: Envío por HTTP

Crea `simulator.py`:

```python
import requests
import time
import random
from datetime import datetime

TOKEN = "TU_ACCESS_TOKEN_AQUI"
URL = f"http://localhost:8080/api/v1/{TOKEN}/telemetry"

print("🚀 Simulador IoT HTTP iniciado...")

while True:
    data = {
        "temperature": round(random.uniform(20, 30), 2),
        "humidity":    round(random.uniform(40, 70), 2),
        "pressure":    round(random.uniform(900, 1100), 2),
        "timestamp":   datetime.now().isoformat()
    }
    try:
        response = requests.post(URL, json=data)
        print(f"[HTTP] Enviado: {data} → Status: {response.status_code}")
    except Exception as e:
        print(f"[ERROR] {e}")
    time.sleep(3)
```

```bash
python3 simulator.py
```

### Opción B: Envío por MQTT

Crea `simulator_mqtt.py`:

```python
import paho.mqtt.client as mqtt
import time
import random
import json
from datetime import datetime

TOKEN  = "TU_ACCESS_TOKEN_AQUI"
BROKER = "localhost"
PORT   = 1883
TOPIC  = "v1/devices/me/telemetry"

client = mqtt.Client()
client.username_pw_set(TOKEN)
client.connect(BROKER, PORT, 60)
client.loop_start()

print("🚀 Simulador MQTT iniciado...")

while True:
    data = {
        "temperature": round(random.uniform(20, 30), 2),
        "humidity":    round(random.uniform(40, 70), 2),
        "pressure":    round(random.uniform(900, 1100), 2),
        "timestamp":   datetime.now().isoformat()
    }
    client.publish(TOPIC, json.dumps(data))
    print(f"[MQTT] Enviado: {data}")
    time.sleep(3)
```

```bash
python3 simulator_mqtt.py
```

---

## 📊 Paso 10 — Visualizar los datos

En ThingsBoard: **Devices** → `Sensor_Prueba_1` → pestaña **Latest Telemetry**

| Clave | Descripción |
|---|---|
| temperature | Temperatura simulada (°C) |
| humidity | Humedad relativa (%) |
| pressure | Presión atmosférica (hPa) |

---

## 🌐 Resumen de acceso a servicios

| Servicio | URL | Puerto |
|---|---|---|
| ThingsBoard (IoT) | `http://IP_SERVIDOR:8080` | 8080 |
| NGINX (Web) | `http://IP_SERVIDOR:8090` | 8090 |

---

## 🚀 Comandos útiles

```bash
# Ver contenedores activos
docker ps

# Levantar servicios en segundo plano
docker compose up -d

# Detener servicios
docker compose down

# Ver logs
docker compose logs -f

# Probar NGINX localmente
curl http://localhost:8090

# Ver qué proceso usa un puerto
sudo lsof -i :8080
sudo lsof -i :8090
```

---

## ♻️ Reinstalar ThingsBoard desde cero

Si necesitas eliminar ThingsBoard completamente y empezar de nuevo:

```bash
# 1. Bajar el servicio y eliminar volúmenes
cd ~/Documents/thingsboard
docker compose down --volumes --remove-orphans

# 2. Eliminar la imagen (opcional, para forzar descarga fresca)
docker rmi thingsboard/tb-postgres

# 3. Verificar que el puerto quedó libre
sudo lsof -i :8080

# 4. Levantar de nuevo (reinstala la BD automáticamente)
docker compose up -d
docker compose logs -f
```

Espera hasta ver en los logs:
```
Installation finished successfully!
Started ThingsboardServerApplication in XX seconds
```

> ⚠️ **ADVERTENCIA — `docker system prune -a --volumes`**  
> Este comando de limpieza de disco **elimina los volúmenes de ThingsBoard**, borrando toda la base de datos.  
> Si lo ejecutas, debes reinstalar ThingsBoard con `docker compose down && docker compose up -d`.  
> Los contenedores activos no se eliminan, pero sus volúmenes de datos sí.

---

## 💾 Configurar Swap (evita congelamiento del sistema)

Sin swap, si ThingsBoard necesita más memoria de la disponible el sistema se congela completamente. Es **obligatorio** configurarlo antes de levantar ThingsBoard.

```bash
# Crear 4 GB de swap
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Hacer el swap permanente (sobrevive reinicios)
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# Verificar que quedó activo
free -h
# Deberías ver: Swap: 4.0Gi
```

> ℹ️ El swap usa el disco como extensión de RAM. Es más lento que la RAM pero evita que el sistema se bloquee cuando la memoria se agota.

---

## ⚠️ Problemas comunes

### ❌ Puerto 8080 ocupado

```bash
sudo lsof -i :8080   # Ver qué lo ocupa
docker stop <nombre_contenedor>  # Liberar el puerto
```

### ❌ No puedo acceder desde el navegador

- Verifica que los dispositivos están en la misma red
- Comprueba que el contenedor está corriendo: `docker ps`
- Revisa los logs: `docker compose logs -f`
- ThingsBoard tarda ~90 segundos en estar listo tras arrancar

### ❌ Error 401 en ThingsBoard

El Access Token es incorrecto. Vuelve a copiarlo desde **Devices → Credentials**.

### ❌ No aparecen datos en Latest Telemetry

- Verifica que el simulador corre sin errores
- Confirma que ThingsBoard está activo: `docker ps`
- Comprueba que la URL del simulador usa el token correcto

### ❌ El sistema se congela completamente (mouse y teclado no responden)

**Causa:** ThingsBoard consumió toda la RAM disponible y no hay swap configurado.

**Solución inmediata:** Reinicio físico — mantén el botón de poder 5-10 segundos.

**Solución definitiva:** Configura swap antes de levantar ThingsBoard (ver sección anterior).

Para monitorear la RAM mientras ThingsBoard arranca:

```bash
# En una terminal separada
watch -n 2 free -h
```

Si `available` baja de 500 MB, ThingsBoard usará swap (más lento pero sin congelar).

### ❌ VS Code Remote-SSH pide contraseña y la rechaza

Configura autenticación por clave SSH para evitar el problema:

```bash
# 1. Generar clave SSH (Enter × 3, sin passphrase)
ssh-keygen -t ed25519 -C "vscode-servidor-iot"

# 2. Copiar la clave al servidor (última vez que pide contraseña)
ssh-copy-id pci@IP_DEL_SERVIDOR

# 3. Probar que entra sin contraseña
ssh pci@IP_DEL_SERVIDOR
```

Después de esto VS Code conectará sin pedir contraseña.

### ❌ Disco lleno — "No space left on device"

```bash
# Ver estado del disco
df -h

# Limpiar imágenes y capas Docker sin usar
# ⚠️ ADVERTENCIA: --volumes elimina la BD de ThingsBoard (ver sección Reinstalar)
docker system prune -a

# Limpiar apt y logs
sudo apt clean
sudo apt autoremove -y
sudo journalctl --vacuum-size=100M

# Ver qué carpetas ocupan más espacio
du -sh /home/pci/* 2>/dev/null | sort -rh | head -10
```

### ❌ "Database error" al entrar a ThingsBoard

Ocurre cuando los volúmenes de la base de datos fueron eliminados (por ejemplo con `docker system prune --volumes`). Solución:

```bash
cd ~/Documents/thingsboard
docker compose down
docker compose up -d
docker compose logs -f
# Espera: "Installation finished successfully!"
```

### ❌ SSH no conecta después de un congelamiento

Si el servidor se congeló completamente, SSH tampoco responderá (`No route to host`). En ese caso es necesario el reinicio físico.

```bash
sudo systemctl status ssh
```

### ❌ Docker no puede descargar imágenes (TLS handshake timeout)

Ocurre cuando Docker no puede resolver DNS correctamente. Solución:

```bash
# Configurar DNS de Google para Docker
echo '{"dns": ["8.8.8.8", "8.8.4.4"]}' | sudo tee /etc/docker/daemon.json
sudo systemctl restart docker

# Reintentar la descarga
docker compose up -d
```

Si el error persiste, verifica que Docker Hub responde:

```bash
curl -I https://registry-1.docker.io
# Respuesta esperada: HTTP/2 404  ← normal, significa que el registry está accesible
```

### ❌ La IP del servidor cambia entre reinicios

Si el servidor usa WiFi con DHCP, la IP puede cambiar. Para verificar la IP actual:

```bash
ip a
# Busca la interfaz activa (wlp4s0 para WiFi, enp*s* para cable)
# La IP está en la línea: inet XXX.XXX.XXX.XXX/21
```

> 💡 **Solución recomendada:** Reserva la IP en el router por MAC address, o configura IP estática en Ubuntu para que la IP nunca cambie.

---

## ✅ Estado del proyecto

- [x] Swap de 4 GB configurado y permanente
- [x] Servidor Ubuntu configurado
- [x] Acceso remoto SSH funcional
- [x] Autenticación SSH por clave configurada (sin contraseña)
- [x] Docker y contenedores activos
- [x] NGINX operativo (puerto 8090) — via docker compose
- [x] ThingsBoard v4.2.1.1 operativo (puerto 8080) — via docker compose (`tb-postgres`)
- [x] Simulación de datos IoT por HTTP
- [x] Simulación de datos IoT por MQTT
- [x] Visualización en dashboards en tiempo real

---

## 🔥 Próximos pasos

- [ ] **Asignar IP estática al servidor** (para que la IP no cambie entre reinicios)
- [ ] Crear dashboards personalizados en ThingsBoard
- [ ] Configurar alarmas y reglas de procesamiento
- [ ] Integrar broker MQTT externo (Mosquitto)
- [ ] Conectar dispositivos reales (ESP32, Arduino)
- [ ] Configurar Reverse Proxy con NGINX
- [ ] Integrar Node-RED para automatización
- [ ] Agregar Grafana para visualizaciones avanzadas
- [ ] Configurar acceso remoto seguro (VPN / túneles)

---

## 👨‍💻 Autor

**Julián Felipe**  
Proyecto desarrollado como entorno de aprendizaje y despliegue de arquitectura IoT local.  
Enfocado en: IoT · Sistemas embebidos · Servidores locales · Docker
