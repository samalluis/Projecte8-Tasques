


# 📘 Informe Técnico: Exploración de Redes con Kali Linux
## RA4.AA4 — Práctica de Laboratorio (Entorno Doméstico)

> **Asignatura:** — Seguretat Informàtica (CFGM SMiX)  
> **Autor:** Lluis Garcia — 2n A, Lista Nº 07  
> **Fecha de elaboración:** Mayo 2026

---

## 🏗️ 1. Preparación del Entorno de Laboratorio

Para llevar a cabo esta práctica de exploración de redes, fue necesario configurar previamente la máquina virtual de Kali Linux con los siguientes parámetros de red:

| Parámetro | Configuración | Descripción |
|-----------|---------------|-------------|
| **Modo de adaptador** | Puente (Bridged) | Permite que la VM comparta la misma red física que el host |
| **Rango de red** | `192.168.2.0/24` | Subred asignada al grupo 2n A |
| **Puerta de enlace** | `192.168.2.254` | Dirección del router VyOS (primera IP del rango) |
| **DNS** | Configuración manual | Resolución de nombres habilitada |

> 💡 **Nota:** Adicionalmente, debían estar operativos tanto la puerta de enlace (VyOS) como el servidor Ubuntu ubicado en el equipo del proyector.

Estas direcciones IP servirán como objetivo para los escaneos que se realizarán a continuación.

![Configuración de red de Kali Linux](<pics/Captura de pantalla 2026-05-18 193528.png>)

---

## 🔍 2. Descubrimiento de Hosts con Netdiscover

Netdiscover es una herramienta de línea de comandos diseñada para el reconocimiento de equipos en redes locales, operando tanto en modo activo (mediante el envío de paquetes ARP) como en modo pasivo (mediante escucha pasiva del tráfico).

---

### 2.1 — Modo Activo: Sondeo ARP Proactivo

En modo activo, la herramienta realiza un sondeo continuo de la red transmitiendo paquetes ARP de solicitud para identificar todos los dispositivos conectados.

| Comando | Descripción |
|---------|-------------|
| `netdiscover -r 192.168.2.0/24` | Escaneo activo del rango completo de la subred |

**Resultados obtenidos:** El sondeo forzó respuestas de todos los equipos conectados en el entorno doméstico, permitiendo identificar sus direcciones IP, direcciones MAC y los fabricantes correspondientes de las interfaces de red (MAC Vendor).

![Resultados del escaneo activo con Netdiscover](<pics/Captura de pantalla 2026-05-18 194357.png>)

---

### 2.2 — Modo Pasivo: Escucha ARP Silenciosa

Posteriormente, se ejecutó el sondeo en modo pasivo, el cual no genera tráfico propio y se limita a capturar el tráfico ARP existente en la red.

| Comando | Descripción |
|---------|-------------|
| `netdiscover -p` | Escaneo pasivo sin generación de paquetes propios |

**Resultados obtenidos:** En este modo no se transmitió ninguna petición ARP; el sistema permaneció en estado de escucha pasiva, capturando únicamente el tráfico ARP que circulaba de forma natural por la red local.

> 💡 **Nota técnica:** Para que aparecieran resultados en el modo pasivo, fue necesario generar tráfico ARP adicional navegando desde un dispositivo móvil conectado a la misma red.

![Resultados del escaneo pasivo con Netdiscover](<pics/Captura de pantalla 2026-05-18 194602.png>)

---

### 2.3 — Análisis Comparativo: Activo vs. Pasivo

| Característica | Modo Activo | Modo Pasivo |
|----------------|-------------|-------------|
| **Generación de tráfico** | Envía peticiones ARP constantes | No genera paquetes propios |
| **Velocidad de descubrimiento** | Muy rápida y completa | Depende del tráfico existente |
| **Visibilidad en la red** | Alto — genera "ruido" detectable | Nulo — totalmente invisible |
| **Detección por IDS/IPS** | Puede disparar alertas en Snort/Suricata | No activa sistemas de detección |
| **Uso recomendado** | Inventario rápido de red | Fase de reconocimiento furtivo |

**Conclusión del análisis:** La diferencia fundamental reside en la generación de tráfico. El sondeo activo emite peticiones ARP de forma continua, lo que lo hace extremadamente rápido y exhaustivo, pero a costa de generar un elevado nivel de actividad en la red. Por el contrario, el sondeo pasivo resulta la opción más adecuada para la fase de reconocimiento previa a un ataque, ya que al no generar paquetes propios, no activa sistemas de detección de intrusos (IDS) como Snort o Suricata, manteniendo una presencia completamente invisible en la red.

---

## 🌐 3. Exploración de Red con Nmap

Nmap (*Network Mapper*) constituye la herramienta de referencia para el escaneo de redes y dispositivos, permitiendo identificar hosts activos y determinar qué puertos y servicios se encuentran expuestos.

---

### 3.1 — Sondeo de Hosts en la Red Doméstica

Se realizó un escaneo de descubrimiento de hosts para verificar qué equipos de la red doméstica respondían a las peticiones, validando así el inventario obtenido previamente con Netdiscover.

| Comando | Descripción |
|---------|-------------|
| `nmap -sn 192.168.0.0/24` | Sondeo de hosts activos sin escaneo de puertos |

![Resultados del sondeo de hosts con Nmap](<pics/Captura de pantalla 2026-05-18 194905.png>)

---

### 3.2 — Análisis Detallado del Router Doméstico

Siguiendo las indicaciones de la práctica para alumnos que la realizan desde casa, se dirigió el análisis detallado exclusivamente hacia el **router doméstico**.

Se empleó la opción de detección de sistema operativo para obtener información completa tanto del sistema como de los servicios y puertos expuestos.

| Comando | Descripción |
|---------|-------------|
| `nmap -O -sV 192.168.2.3` | Detección de SO y versiones de servicios (VyOS) |
| `nmap -O -sV 192.168.2.8` | Detección de SO y versiones de servicios (Ubuntu) |

**Hallazgos del análisis:**

A partir del escaneo realizado con Nmap sobre las direcciones IP del router VyOS (`192.168.2.3`) y del servidor Ubuntu (`192.168.2.8`), se detectó que **ninguno de los dos dispositivos presenta puertos abiertos** dentro de los 1000 puertos TCP más comunes analizados, ya que todos aparecen en estado cerrado (reset). Puertos habitualmente activos como el 22 (SSH), 23 (Telnet) o 443 (HTTPS) se encuentran completamente inactivos en ambos equipos.

Respecto a la identificación del sistema operativo, Nmap no proporcionó una coincidencia exacta (mensaje: *"Too many fingerprints match"*) debido a la ausencia de puertos abertos con los que interactuar. No obstante, la herramienta detectó claramente que la dirección MAC de ambos dispositivos pertenece al fabricante **Oracle VirtualBox virtual NIC**, evidenciando de forma inequívoca que se trata de entornos virtualizados.

![Resultados del escaneo detallado con Nmap](<pics/Captura de pantalla 2026-05-18 195827.png>)

---

## 📝 4. Conclusiones y Aprendizajes Obtenidos

A lo largo de esta práctica de exploración de redes con Kali Linux, se han consolidado diversos conocimientos y competencias técnicas fundamentales en el ámbito de la ciberseguridad y el análisis de redes:

### 🔑 Competencias Desarrolladas

| Área de conocimiento | Aprendizaje clave |
|----------------------|-------------------|
| **Configuración de red** | Comprensión de la importancia de la correcta configuración del adaptador en modo puente para que la VM comparta la subred del host físico |
| **Reconocimiento activo** | Dominio de Netdiscover en modo activo para el descubrimiento rápido de equipos mediante peticiones ARP, identificando direcciones IP, MAC y fabricantes |
| **Reconocimiento pasivo** | Aplicación de técnicas de escucha pasiva para obtener información sin generar tráfico propio, esencial para operaciones de reconocimiento furtivo |
| **Análisis comparativo** | Capacidad para evaluar las ventajas e inconvenientes de cada método de sondeo según el contexto operativo |
| **Escaneo con Nmap** | Uso de Nmap para el descubrimiento de hosts (`-sn`) y el análisis detallado de sistemas (`-O -sV`), interpretando correctamente los resultados |
| **Interpretación de resultados** | Comprensión de estados de puertos (abierto, cerrado, filtrado) y limitaciones en la detección de sistemas operativos cuando no hay servicios activos |
| **Identificación de virtualización** | Reconocimiento de entornos virtualizados a través del análisis de direcciones MAC y fabricantes de interfaces de red |

### 🎯 Reflexión Final

Esta práctica ha permitido comprender de primera mano la importancia de las **fases iniciales de reconocimiento** en cualquier auditoría de seguridad. La elección entre técnicas activas y pasivas no es meramente técnica, sino que responde a una decisión estratégica: el sondeo activo prioriza la velocidad y exhaustividad, mientras que el pasivo maximiza la stealth y evita la detección por sistemas IDS/IPS.

Asimismo, los resultados obtenidos con Nmap evidencian que la ausencia de puertos abiertos no implica necesariamente una red segura, sino que puede deberse a configuraciones de firewall o a que los servicios escuchan en puertos no estándar. La capacidad de interpretar correctamente los resultados del escaneo —incluyendo las limitaciones como la imposibilidad de identificar el SO por falta de huellas activas— constituye una habilidad crítica para cualquier profesional de ciberseguridad.

Finalmente, la detección de entornos virtualizados a través del análisis de direcciones MAC demuestra que incluso la información aparentemente trivial puede revelar detalles significativos sobre la infraestructura objetivo, reforzando la necesidad de una metodología de reconocimiento sistemática y minuciosa.

---

> 📋 **Documento técnico** | Exploración de Redes con Kali Linux | Mòdul 0226 Seguretat Informàtica  
> [← Volver al índice](README.md)
"""

