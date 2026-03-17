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
