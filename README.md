# 🖥️ Servidor Local con Ubuntu + Docker Compose

## 📌 Descripción del proyecto

Este proyecto consiste en la configuración de un **servidor local** sobre una mini-PC utilizando **Ubuntu 22.04.5 LTS** y **Docker Compose**, con el objetivo de desplegar servicios de manera modular, escalable y reproducible.

---

# 🧠 1. ¿Qué es un servidor?

Un **servidor** es un sistema que proporciona servicios a otros dispositivos (clientes) dentro de una red.

### Ejemplos de servicios:

* Servidor web (nginx, Apache)
* Servidor IoT (MQTT, Node-RED)
* Base de datos
* Sistemas distribuidos (ROS2)

---

# 🐧 2. Sistema Operativo: Ubuntu 22.04.5 LTS

### ¿Qué significa LTS?

**LTS (Long Term Support)** garantiza:

* Soporte por 5 años
* Estabilidad
* Actualizaciones de seguridad

Ideal para servidores.

---

# 🔄 3. Actualización del sistema

Después de instalar Ubuntu, es fundamental actualizar los paquetes.

```bash
sudo apt update
sudo apt upgrade -y
```

### Explicación:

* `apt update`: actualiza la lista de paquetes disponibles
* `apt upgrade`: instala actualizaciones

---

# 🌐 4. Verificación de red

Para identificar la IP del servidor:

```bash
ip a
```

Buscar:

```
inet 192.168.x.x
```

### ¿Qué es una IP?

Es la dirección única que identifica un dispositivo en la red.

---

# 🔐 5. Acceso remoto con SSH

SSH permite controlar el servidor de forma remota.

## Instalación:

```bash
sudo apt install openssh-server -y
```

## Verificación:

```bash
sudo systemctl status ssh
```

## Conexión desde otro equipo:

```bash
ssh usuario@IP_SERVIDOR
```

---

# 🐳 6. Docker

## ¿Qué es Docker?

Docker es una plataforma que permite ejecutar aplicaciones en **contenedores**.

### Contenedor:

Un entorno aislado que incluye:

* aplicación
* dependencias
* configuración

---

## Instalación (si no está instalado)

```bash
sudo apt install docker.io -y
```

## Verificar instalación

```bash
docker --version
```

---

## Activar servicio

```bash
sudo systemctl enable docker
sudo systemctl start docker
```

---

## Ejecutar prueba

```bash
docker run hello-world
```

---

# ⚙️ 7. Docker Compose

## ¿Qué es Docker Compose?

Permite definir y ejecutar múltiples contenedores usando un archivo YAML.

---

## Verificar instalación

```bash
docker compose version
```

## Instalación (si no está disponible)

```bash
sudo apt install docker-compose-plugin -y
```

---

# 🔓 8. Ejecutar Docker sin sudo

```bash
sudo usermod -aG docker $USER
newgrp docker
```

---

# 📁 9. Estructura del servidor

Se recomienda organizar los servicios:

```bash
mkdir -p ~/server/nginx
cd ~/server/nginx
```

Estructura sugerida:

```
server/
 ├── nginx/
 ├── iot/
 ├── db/
 └── monitoring/
```

---

# 🌍 10. Primer servicio: NGINX

## Crear archivo docker-compose.yml

```bash
nano docker-compose.yml
```

## Contenido:

```yaml
services:
  nginx:
    image: nginx:latest
    container_name: nginx_server
    ports:
      - "8080:80"
    restart: unless-stopped
```

---

## Explicación:

* `image`: imagen base
* `container_name`: nombre del contenedor
* `ports`: mapeo de puertos (host:contenedor)
* `restart`: política de reinicio

---

# ▶️ 11. Levantar el servicio

```bash
docker compose up -d
```

### Parámetro:

* `-d`: modo background

---

# 🔍 12. Verificar contenedores

```bash
docker ps
```

---

# 🌐 13. Acceso al servicio

Desde navegador:

```
http://IP_SERVIDOR:8080
```

---

# 📊 14. Comandos útiles

## Ver contenedores activos

```bash
docker ps
```

## Ver todos los contenedores

```bash
docker ps -a
```

## Detener servicios

```bash
docker compose down
```

## Ver logs

```bash
docker compose logs
```

---

# 🧱 15. Ventajas de Docker Compose

* Aislamiento de servicios
* Escalabilidad
* Reproducibilidad
* Portabilidad
* Fácil mantenimiento

---

# 🚀 16. Próximos pasos

Posibles extensiones del servidor:

* MQTT (Mosquitto)
* Node-RED
* InfluxDB + Grafana
* ThingsBoard
* ROS2 distribuido

---

# 👨‍💻 Autor

Proyecto desarrollado por Julián Felipe
Enfocado en sistemas embebidos, IoT y servidores locales.

---
