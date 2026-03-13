# BeaglePlay ThingsBoard Gateway (WSL + MQTT)

## Descripción

Este proyecto implementa un **Gateway de ThingsBoard** que se ejecuta en un dispositivo **BeaglePlay** (simulado inicialmente en WSL).

El gateway permitirá conectar dispositivos IoT como **ESP32** a la plataforma **ThingsBoard** utilizando **MQTT**.

El objetivo es centralizar la comunicación de múltiples dispositivos IoT mediante un gateway intermedio.

---

# Arquitectura del sistema

La arquitectura general del sistema es la siguiente:

```
ESP32 → Mosquitto Broker → BeaglePlay Gateway → ThingsBoard
```

Donde:

* **ESP32**: Dispositivo IoT que envía telemetría.
* **Mosquitto**: Broker MQTT que recibe los mensajes.
* **BeaglePlay**: Ejecuta el ThingsBoard Gateway.
* **ThingsBoard**: Plataforma IoT para visualización y gestión de dispositivos.

---

# 1. Preparación del entorno Linux

Se utiliza **WSL (Windows Subsystem for Linux)** para trabajar con un entorno Linux dentro de Windows.

Verificar versión de Python:

```bash
python3 --version
```

Instalar dependencias necesarias:

```bash
sudo apt update
sudo apt install python3 python3-pip python3-venv
```

---

# 2. Crear directorio del proyecto

```bash
mkdir ~/InvesIoT
cd ~/InvesIoT
```

Crear carpeta para la configuración del gateway:

```bash
mkdir tb-gateway-config
cd tb-gateway-config
```

---

# 3. Crear entorno virtual de Python

Debido a la restricción **PEP 668 (externally-managed-environment)** se debe usar un entorno virtual.

Crear entorno virtual:

```bash
python3 -m venv tb-gateway-env
```

Activar entorno virtual:

```bash
source tb-gateway-env/bin/activate
```

Cuando el entorno esté activo verás algo así en la terminal:

```
(tb-gateway-env)
```

---

# 4. Instalar ThingsBoard Gateway

Instalar el gateway usando pip:

```bash
pip install thingsboard-gateway
```

Verificar instalación:

```bash
pip list | grep tb
```

Salida esperada:

```
tb-mqtt-client
tb-paho-mqtt-client
thingsboard-gateway
```

---

# 5. Crear Gateway en ThingsBoard

Ingresar a la plataforma:

```
https://thingsboard.cloud
```

Luego ir a:

```
Devices → Add Device
```

Configurar:

Nombre del dispositivo:

```
BeaglePlay-Gateway
```

Activar la opción:

```
Is Gateway ✓
```

Guardar y copiar el **Access Token** generado.

Ejemplo:

```
yMf6LyV37kY4nFMAeuER
```

---

# 6. Crear archivo de configuración del gateway

Crear el archivo:

```bash
nano tb_gateway.json
```

Contenido del archivo:

```json
{
  "thingsboard": {
    "host": "thingsboard.cloud",
    "port": 1883,
    "security": {
      "accessToken": "YOUR_ACCESS_TOKEN"
    }
  },
  "storage": {
    "type": "memory"
  },
  "connectors": [],
  "grpc": {
    "enabled": false
  }
}
```

Reemplazar:

```
YOUR_ACCESS_TOKEN
```

por el token generado en ThingsBoard.

---

# 7. Ejecutar el gateway

Iniciar el gateway con:

```bash
python -m thingsboard_gateway.tb_gateway -c tb_gateway.json
```

Salida esperada en consola:

```
Gateway starting...
ThingsBoard IoT gateway version: 3.8.2
Connecting to ThingsBoard...
MQTT client connected to platform
Gateway connected
```

---

# 8. Verificar conexión en ThingsBoard

Ir a:

```
ThingsBoard → Devices
```

Seleccionar:

```
BeaglePlay-Gateway
```

El estado debe aparecer como:

```
Active
```

---

# 9. Advertencias normales en el log

Es normal ver el siguiente mensaje:

```
Connectors - not found, waiting for remote configuration
```

Esto significa que **aún no se ha configurado ningún conector**.

Posteriormente se agregará un **MQTT Connector** para recibir datos desde Mosquitto.

---

# 10. Próximos pasos

Las siguientes etapas del proyecto incluyen:

1. Instalar **Mosquitto MQTT Broker**
2. Configurar el **MQTT Connector** en ThingsBoard Gateway
3. Conectar dispositivos **ESP32**
4. Enviar telemetría a ThingsBoard
5. Crear dashboards de visualización

Arquitectura final del sistema:

```
ESP32 → MQTT (Mosquitto) → BeaglePlay Gateway → ThingsBoard Dashboard
```

---

# Tecnologías utilizadas

* ESP32
* BeaglePlay
* ThingsBoard
* MQTT
* Mosquitto
* Python
* WSL (Linux)

---

# Autor

**Julian Felipe**

Proyecto IoT basado en arquitectura de gateway para integración de dispositivos embebidos con plataformas IoT.

---

Si quieres, también puedo ayudarte a hacer una **versión aún más profesional del README para GitHub** con:

* badges
* diagramas de arquitectura
* estructura de carpetas del proyecto
* guía para **ESP32 enviando datos a ThingsBoard**
* dashboards de ejemplo.
