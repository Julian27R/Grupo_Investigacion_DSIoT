# 🖥️ Servidor IoT Local con Ubuntu, Docker y ThingsBoard

> Proyecto de aprendizaje para montar un servidor local completo con capacidades IoT, usando un mini-PC, contenedores Docker y la plataforma ThingsBoard.

---

## 📌 ¿Qué hace este proyecto?

Este proyecto te guía paso a paso para convertir un mini-PC con Ubuntu en un **servidor local IoT** capaz de:

- Recibir datos de sensores (reales o simulados)
- Visualizarlos en dashboards en tiempo real
- Ser accesible desde cualquier dispositivo en la misma red

No necesitas experiencia previa — cada sección explica el concepto antes de ejecutar comandos.

---

## 🧠 Conceptos clave (léelos antes de empezar)

### ¿Qué es un servidor?
Un servidor es un computador que ofrece servicios a otros dispositivos en la red. En este proyecto, tu mini-PC será el servidor.

### ¿Qué es Docker?
Docker permite "empaquetar" programas en **contenedores** — como cajas selladas que incluyen todo lo necesario para que una aplicación funcione, sin afectar el resto del sistema.

### ¿Qué es IoT?
IoT (Internet of Things) es la red de dispositivos físicos (sensores, microcontroladores) que envían datos a través de internet o redes locales.

### ¿Qué es ThingsBoard?
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
| ThingsBoard Edge | Plataforma IoT |
| NGINX | Servidor web |
| Python 3 | Simulador de dispositivos IoT |
| SSH / VS Code Remote | Acceso remoto al servidor |

---

## 🖥️ Paso 1 — Preparar Ubuntu

Ubuntu 22.04 LTS es la versión recomendada para servidores por su estabilidad y 5 años de soporte oficial.

### Actualizar el sistema

Siempre empieza actualizando los paquetes instalados:
```bash
sudo apt update
sudo apt upgrade -y
```

### Identificar la IP del servidor

La dirección IP es el identificador del servidor dentro de tu red local. La necesitarás para conectarte desde otros dispositivos.
```bash
ip a
```

Busca una línea como esta:
```
inet 172.20.25.173/21
```

> 📌 Anota esta IP — la usarás en todos los pasos siguientes.

---

## 🔐 Paso 2 — Acceso remoto con SSH

SSH te permite controlar el servidor desde otro computador sin necesidad de teclado ni monitor conectados.

### Instalar SSH
```bash
sudo apt install openssh-server -y
```

### Verificar que está activo
```bash
sudo systemctl status ssh
```

Debe decir `active (running)`.

### Conectarse desde otro equipo
```bash
ssh tu_usuario@IP_DEL_SERVIDOR
# Ejemplo:
ssh pci@172.20.25.173
```

---

## 💻 Paso 3 (opcional) — Conectar VS Code al servidor

VS Code permite editar archivos directamente en el servidor de forma visual.

1. Instala la extensión **Remote - SSH** en VS Code
2. Edita el archivo de configuración SSH:
```bash
nano ~/.ssh/config
```

Agrega esto:
```
Host servidor-iot
    HostName 172.20.25.173
    User pci
```

3. En VS Code: `Ctrl + Shift + P` → `Remote-SSH: Connect to Host` → selecciona `servidor-iot`

---

## 🐳 Paso 4 — Instalar Docker y Docker Compose

Docker es el motor que ejecutará todos los servicios del servidor como contenedores aislados.

### Verificar si Docker ya está instalado
```bash
docker --version
docker compose version
```

### Instalar Docker Compose (si no está disponible)
```bash
sudo apt install docker-compose-plugin -y
```

### Probar que Docker funciona
```bash
docker run hello-world
```

Si ves un mensaje de bienvenida, Docker está funcionando correctamente.

---

## 📁 Paso 5 — Crear la estructura de carpetas

Organizamos los servicios en carpetas separadas:
```bash
mkdir -p ~/server/nginx
cd ~/server/nginx
```

---

## 🌍 Paso 6 — Levantar NGINX (servidor web)

NGINX es un servidor web liviano. Lo usaremos como punto de entrada web en el puerto 8090.

### Crear el archivo `docker-compose.yml`
```bash
nano ~/server/nginx/docker-compose.yml
```

Contenido del archivo:
```yaml
services:
  nginx:
    image: nginx:latest
    container_name: nginx_web_server
    ports:
      - "8090:80"
    restart: unless-stopped
```

> `"8090:80"` significa: el puerto 80 del contenedor se expone como 8090 en el servidor.

### Levantar el servicio
```bash
docker compose up -d
```

La bandera `-d` significa "en segundo plano" (detached).

### Verificar que está corriendo
```bash
docker ps
```

Deberías ver algo así:
```
CONTAINER ID   IMAGE          PORTS                  NAMES
xxxxxxxxxxxx   nginx:latest   0.0.0.0:8090->80/tcp   nginx_web_server
```

---

## 📡 Paso 7 — Desplegar ThingsBoard (plataforma IoT)

ThingsBoard recibirá los datos de los sensores y los mostrará en dashboards.

### Descargar la imagen
```bash
docker pull thingsboard/tb-edge
```

### Ejecutar el contenedor
```bash
docker run -it -p 8080:8080 -p 1883:1883 -p 5683:5683 \
  --name tb-edge \
  thingsboard/tb-edge
```

Los puertos expuestos son:
- `8080` → interfaz web
- `1883` → protocolo MQTT
- `5683` → protocolo CoAP

### Acceder desde el navegador
```
http://IP_DEL_SERVIDOR:8080
# Ejemplo:
http://172.20.25.173:8080
```

### Credenciales por defecto
```
Usuario:    tenant@thingsboard.org
Contraseña: tenant
```

> ⚠️ Cambia la contraseña en producción.

---

## 🔑 Paso 8 — Crear un dispositivo IoT en ThingsBoard

Antes de enviar datos, debes registrar el dispositivo:

1. Entra a ThingsBoard → sección **Devices**
2. Haz clic en **Add Device**
3. Asigna un nombre, por ejemplo: `Sensor_Prueba_1`
4. Entra al dispositivo creado → pestaña **Credentials**
5. Copia el **Access Token** — lo necesitarás en el simulador

---

## 🧪 Paso 9 — Simular un dispositivo IoT

Simulamos un sensor que envía temperatura, humedad y presión cada 3 segundos.

### Instalar dependencias de Python
```bash
pip3 install requests paho-mqtt
```

---

### Opción A: Envío por HTTP (más sencillo)

Crea el archivo `simulator.py`:
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

Ejecutar:
```bash
python3 simulator.py
```

Salida esperada:
```
[HTTP] Enviado: {'temperature': 24.5, 'humidity': 55.2, ...} → Status: 200
```

---

### Opción B: Envío por MQTT (protocolo profesional IoT)

MQTT es un protocolo ligero diseñado específicamente para IoT, más eficiente que HTTP para envíos frecuentes.

Crea el archivo `simulator_mqtt.py`:
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

Ejecutar:
```bash
python3 simulator_mqtt.py
```

---

## 📊 Paso 10 — Visualizar los datos

En ThingsBoard:

1. Ve a **Devices** → `Sensor_Prueba_1`
2. Abre la pestaña **Latest Telemetry**

Verás los datos actualizándose en tiempo real:

| Clave | Descripción |
|---|---|
| `temperature` | Temperatura simulada (°C) |
| `humidity` | Humedad relativa (%) |
| `pressure` | Presión atmosférica (hPa) |

---

## 🌐 Resumen de acceso a servicios

| Servicio | URL | Puerto |
|---|---|---|
| ThingsBoard (IoT) | `http://IP_SERVIDOR:8080` | 8080 |
| NGINX (Web) | `http://IP_SERVIDOR:8090` | 8090 |

Desde cualquier dispositivo en la misma red puedes abrir estas URLs en el navegador.

---

## 🚀 Comandos útiles
```bash
# Ver contenedores activos
docker ps

# Levantar servicios en segundo plano
docker compose up -d

# Detener servicios
docker compose down

# Ver logs de los servicios
docker compose logs

# Probar que NGINX responde localmente
curl http://localhost:8090

# Ver qué proceso usa un puerto
sudo lsof -i :8080
sudo lsof -i :8090
```

---

## ⚠️ Problemas comunes

### ❌ Error: `port is already allocated`
Otro proceso ya usa ese puerto.
```bash
sudo lsof -i :8080   # Ver qué lo ocupa
```

Soluciones:
- Cambiar el puerto en el `docker-compose.yml`
- Detener el contenedor que ocupa el puerto: `docker stop <nombre_contenedor>`

---

### ❌ No puedo acceder desde otro dispositivo
Posibles causas:
- Los dispositivos están en redes diferentes (WiFi vs cableado)
- La red tiene restricciones (redes universitarias, hotspots)
- El firewall del servidor bloquea los puertos

---

### ❌ Error 401 en ThingsBoard
El Access Token es incorrecto. Vuelve a copiarlo desde **Devices → Credentials**.

---

### ❌ No aparecen datos en Latest Telemetry
- Verifica que el simulador esté corriendo sin errores
- Confirma que ThingsBoard está activo: `docker ps`
- Comprueba que la URL del simulador sea correcta

---

### ❌ SSH no conecta
- Verifica que el servicio SSH está activo: `sudo systemctl status ssh`
- Confirma que el usuario y la IP son correctos

---

## ✅ Estado del proyecto

- [x] Servidor Ubuntu configurado
- [x] Acceso remoto SSH funcional
- [x] Docker y contenedores activos
- [x] NGINX operativo (puerto 8090)
- [x] ThingsBoard operativo (puerto 8080)
- [x] Simulación de datos IoT por HTTP
- [x] Simulación de datos IoT por MQTT
- [x] Visualización en dashboards en tiempo real

---

## 🔥 Próximos pasos

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
