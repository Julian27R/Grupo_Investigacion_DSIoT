# 🖥️ Servidor Local con Ubuntu + Docker Compose

## 📌 Descripción del proyecto

Configuración de un **servidor local en mini-PC** usando Ubuntu 22.04.5 LTS y Docker Compose, orientado a servicios IoT y web.

---

# 🧠 1. ¿Qué es un servidor?

Un servidor es un sistema que proporciona servicios a otros dispositivos en una red.

Ejemplos:

* Web (nginx)
* IoT (ThingsBoard, MQTT)
* Bases de datos

---

# 🐧 2. Sistema Operativo

Ubuntu 22.04.5 LTS

**LTS (Long Term Support):**

* 5 años de soporte
* Alta estabilidad
* Ideal para servidores

---

# 🔄 3. Actualización del sistema

```bash
sudo apt update
sudo apt upgrade -y
```

---

# 🌐 4. Identificación de red

```bash
ip a
```

Buscar una línea como:

```text
inet 172.20.25.173/21
```

👉 Esta es la IP del servidor.

---

## 🧠 Concepto: Dirección IP

Es el identificador único de un dispositivo dentro de una red.

Ejemplo:

```text
Servidor → 172.20.25.173
```

---

# 🔐 5. Acceso remoto (SSH)

```bash
sudo apt install openssh-server -y
```

Verificar:

```bash
sudo systemctl status ssh
```

Conexión desde otro equipo:

```bash
ssh usuario@IP_SERVIDOR
```

---

# 🐳 6. Docker

Verificar instalación:

```bash
docker --version
```

Prueba:

```bash
docker run hello-world
```

---

# ⚙️ 7. Docker Compose

Verificar:

```bash
docker compose version
```

Instalar si es necesario:

```bash
sudo apt install docker-compose-plugin -y
```

---

# 📁 8. Estructura del servidor

```bash
mkdir -p ~/server/nginx
cd ~/server/nginx
```

---

# 🌍 9. Servicio Web con NGINX

## docker-compose.yml

```yaml
services:
  nginx:
    image: nginx:latest
    container_name: nginx_web_server
    ports:
      - "8090:80"
    restart: unless-stopped
```

---

## ▶️ Levantar servicio

```bash
docker compose up -d
```

---

## 🔍 Verificar contenedores

```bash
docker ps
```

Debe aparecer:

```text
nginx_web_server → 0.0.0.0:8090->80/tcp
thingsboard-edge → 0.0.0.0:8080->8080/tcp
```

---

# 🌐 10. Acceso a servicios

## ThingsBoard

```text
http://IP_SERVIDOR:8080
```

## NGINX

```text
http://IP_SERVIDOR:8090
```

---

# 🔥 11. Prueba desde otros dispositivos

Desde celular o laptop en la misma red:

```text
http://172.20.25.173:8080
http://172.20.25.173:8090
```

---

# ⚠️ 12. Solución de problemas

## 🔸 Verificar servicio local

```bash
curl http://localhost:8090
```

---

## 🔸 Verificar puertos ocupados

```bash
sudo lsof -i :8080
sudo lsof -i :8090
```

---

## 🔸 Ver contenedores activos

```bash
docker ps
```

---

## 🔸 Problemas comunes

### ❌ Puerto ocupado

Error:

```text
port is already allocated
```

Solución:

* Cambiar puerto
* Detener contenedor existente

---

### ❌ No abre desde otro dispositivo

Posibles causas:

* Firewall
* Red diferente
* Hotspot o red restringida

---

# 🧠 13. Arquitectura actual

```text
Mini-PC (Servidor)
│
├── Puerto 8080 → ThingsBoard (IoT)
└── Puerto 8090 → nginx (web)
```

---

# 🚀 14. Comandos útiles

```bash
docker ps
docker compose up -d
docker compose down
docker compose logs
```

---

# 🔥 15. Próximos pasos

* Reverse proxy con nginx
* Integración con Node-RED
* MQTT (Mosquitto)
* Dashboards (Grafana)
* Arquitectura IoT completa

---

# 👨‍💻 Autor

Julián Felipe
Proyecto enfocado en IoT, sistemas embebidos y servidores locales.

---

# 🖥️ Servidor IoT Local con Ubuntu, Docker y ThingsBoard

## 📌 Descripción del Proyecto

Este proyecto consiste en la implementación de un **servidor local IoT** utilizando un mini-PC con **Ubuntu 22.04 LTS**, contenedores Docker y la plataforma ThingsBoard para la gestión y visualización de datos.

El sistema permite simular dispositivos IoT que envían datos en tiempo real y visualizar dichos datos mediante dashboards.

---

## 🧠 1. Arquitectura del Sistema

```
[Simulador IoT (Python)]
            │
            ▼
     (HTTP / MQTT)
            │
            ▼
   ThingsBoard (Docker)
            │
            ▼
      Dashboard Web
```

---

## 🖥️ 2. Sistema Operativo

* Ubuntu 22.04.5 LTS
* Soporte a largo plazo (5 años)
* Estable y optimizado para servidores

### 🔄 Actualización del sistema

```bash
sudo apt update
sudo apt upgrade -y
```

---

## 🌐 3. Configuración de Red

Identificar la IP del servidor:

```bash
ip a
```

Ejemplo:

```
inet 172.20.28.165/21
```

---

## 🔐 4. Acceso Remoto (SSH)

### Instalación:

```bash
sudo apt install openssh-server -y
```

### Verificación:

```bash
sudo systemctl status ssh
```

### Conexión:

```bash
ssh pci@172.20.28.165
```

---

## 💻 5. Integración con VS Code

Se utiliza **Remote - SSH** para trabajar directamente sobre el servidor.

### Configuración SSH:

Editar archivo:

```bash
nano ~/.ssh/config
```

Agregar:

```
Host servidor-iot
    HostName 172.20.28.165
    User pci
```

### Conexión desde VS Code:

* `Ctrl + Shift + P`
* `Remote-SSH: Connect to Host`
* Seleccionar: `servidor-iot`

---

## 🐳 6. Docker

### Verificación:

```bash
docker --version
```

### Prueba:

```bash
docker run hello-world
```

---

## ⚙️ 7. Docker Compose

### Verificación:

```bash
docker compose version
```

### Instalación:

```bash
sudo apt install docker-compose-plugin -y
```

---

## 📁 8. Estructura del Proyecto

```bash
~/server/nginx
```

---

## 🌍 9. Servidor Web con NGINX

### docker-compose.yml

```yaml
services:
  nginx:
    image: nginx:latest
    container_name: nginx_web_server
    ports:
      - "8090:80"
    restart: unless-stopped
```

### Levantar servicio:

```bash
docker compose up -d
```

### Ver contenedores:

```bash
docker ps
```

---

## 📡 10. Plataforma IoT (ThingsBoard)

Acceso desde navegador:

```
http://172.20.28.165:8080
```

### Credenciales por defecto:

```
usuario: tenant@thingsboard.org
contraseña: tenant
```

---

## 🔑 11. Creación de Dispositivo IoT

1. Ir a **Devices**
2. Click en **Add Device**
3. Asignar nombre (ej: Sensor_Prueba)
4. Copiar **Access Token**

---

## 🧪 12. Simulador IoT (Python)

### Crear proyecto:

```bash
mkdir ~/iot-project
cd ~/iot-project
```

### Archivo: `simulator.py`

```python
import requests
import random
import time

THINGSBOARD_HOST = "http://localhost:8080"
ACCESS_TOKEN = "TU_TOKEN_AQUI"

url = f"{THINGSBOARD_HOST}/api/v1/{ACCESS_TOKEN}/telemetry"

while True:
    data = {
        "temperature": round(random.uniform(20, 30), 2),
        "humidity": round(random.uniform(40, 70), 2)
    }

    try:
        response = requests.post(url, json=data)
        print("Enviado:", data, "Status:", response.status_code)
    except Exception as e:
        print("Error:", e)

    time.sleep(5)
```

---

## 📦 13. Instalación de dependencias

```bash
pip3 install requests
```

---

## ▶️ 14. Ejecución

```bash
python3 simulator.py
```

---

## 📊 15. Visualización de Datos

En ThingsBoard:

* Devices → Sensor_Prueba
* Latest Telemetry

Datos esperados:

* temperature
* humidity

---

## 🌐 16. Acceso a Servicios

| Servicio    | URL                       |
| ----------- | ------------------------- |
| ThingsBoard | http://172.20.28.165:8080 |
| NGINX       | http://172.20.28.165:8090 |

---

## ⚠️ 17. Problemas Comunes

### ❌ No conecta SSH

* Verificar usuario correcto
* Verificar servicio SSH activo

### ❌ No abre en red

* Dispositivos en diferente red
* Restricciones de red (universidad)

### ❌ Puerto ocupado

```bash
sudo lsof -i :8080
sudo lsof -i :8090
```

---

## 🧠 18. Arquitectura Actual

```
Mini-PC (Servidor)
│
├── Puerto 8080 → ThingsBoard
└── Puerto 8090 → NGINX
```

---

## 🚀 19. Próximos Pasos

* Integración con MQTT (Mosquitto)
* Node-RED para automatización
* Reverse Proxy con NGINX
* Dashboards avanzados
* Acceso remoto (VPN / túneles)

---

## ✅ Estado del Proyecto

✔️ Servidor Ubuntu configurado
✔️ Acceso remoto SSH funcional
✔️ Docker y contenedores activos
✔️ ThingsBoard operativo
✔️ Simulación de datos IoT en tiempo real
✔️ Visualización en dashboards

---

## 👨‍💻 Autor

Proyecto desarrollado como entorno de aprendizaje y despliegue de arquitectura IoT local.

