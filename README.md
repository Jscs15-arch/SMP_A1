# *SMP*_A1
## ***S***ervidor ***M***ultipropósito ***P***ersonal

## 1. Especificaciones utilizadas para el servidor

> [!Note]
> No es necesario que sean las mismas.

| Especificación | Valor |
|----------------|-------|
| **CPU** | Ryzen 5 Pro 6650H (6C/12T) |
| **RAM** | 16 GB DDR5 |
| **Almacenamiento** | 1 TB SSD |
| **S.O.** | Ubuntu Server 24.04.3. |

## 2. Configuraciones básicas del servidor

### 2.1. Configuración de red

Para esto tenemos 2 opciones:

#### 2.1.1. Utilizar el servidor DHCP del router

Con esto, solo debemos asegurarnos de saber cuál es el puerto de conexión utilizado y dejar que DHCP configure la interfaz.

#### 2.1.2. Configuración manual

Al configurar manualmente debemos conocer que IP's estan disponibles, cual es la red o subred en la que nos encontramos y Gateway (puerta de enlace), ademas de otras opciones adicionales como DNS

**Ejemplo:**
| Opción | Valor |
|--------|-------|
| **IP** | 192.168.1.100 |
| **Red** | 192.168.1.0/24 |
| **Gateway** | 192.168.1.1 |
| **DNS** | 1.1.1.1, 8.8.8.8 |

> [!Note]
> Para este ejemplo, he usado una IP aleatoria dentro de la red definida, como gateway la común en redes domésticas y, de DNS, Google y Cloudflare.

### 2.2. Almacenamiento

En mi caso lo he personalizado de la siguiente forma:

| Volumen | Tamaño |
|---------|--------|
| **/** | 100 GB |
| **swap** | Automatica |
| **/data** | Resto del disco |

### 2.3. Servidor **SSH**

Para **SSH**, podemos instalarlo durante la instalación del servidor en uno de los pasos del asistente o de la siguiente manera a través de la CLI:

```
sudo apt update
sudo apt install ssh
```
Revisar que el servicio se encuentra activado.

```bash
systemctl status ssh
```
Salida:
```console
ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/usr/lib/systemd/system/ssh.service; disabled; preset: enabled)    // Como vemos el preset se encuentra enabled
          Active: active (running) since Tue 2026-02-24 18:02:45 CET; 16h ago           // Nos centramos en esta linea
          TriggeredBy: ● ssh.socket
```

Si este servicio aparece como desactivado, faltaría activarlo; y si quieres que siempre esté activo al arrancar, utiliza enable.

```bash
sudo systemctl start ssh
sudo system enable ssh
```

## 3. Servicios a utilizar

Primero instalaremos los servicios más críticos y necesarios, además de algunos que provee Ubuntu; luego iremos con los referentes a los usuarios y necesidades.

> [!Important]
> No todos son necesarios

| Servicio | Uso | En Sistema/Contenedor | Descripción |
|----------|-----|-----------------------|-------------|
| **Netbird** | *VPN* | Sistema |  Plataforma de código abierto diseñada para crear redes privadas virtuales seguras y de alto rendimiento basadas en el protocolo WireGuard, Una de sus características son sus Redes Mesh Peer-to-Peer que le permite conectar todos los equipos directamente entre sí |
| **UFW** | *Firewall* | Sistema | ***U***ncomplicated ***F***ire***W***all es una herramienta de seguridad diseñada para gestionar de forma sencilla el cortafuegos en sistemas Linux. |
| **Fail2ban** | *IPS* | Sistema | Herramienta de seguridad esencial para servidores Linux que previene ataques de fuerza bruta. |
| **Apache2** | *Reverse proxy* | Sistema | Como proxy inverso, el servidor deja de ser solo un "despachador" de archivos para convertirse en un intermediario inteligente entre internet y las aplicaciones internas. |
| **Webmin** | *Monitoreo y Otros* | Sistema |  Interfaz gráfica basada en web para la administración de sistemas Linux y Unix, diseñada para gestionar el servidor desde el navegador sin necesidad de editar archivos de configuración manualmente en la terminal. |
| **Docker** | *Contenerizado* | Sistema | Plataforma líder de contenerización que permite empaquetar una aplicación y todas sus dependencias (bibliotecas, código, configuración) en una unidad estandarizada llamada contenedor. |
| **Portainer** | *Monitor de Contenedores* | Contenedor | Interfaz gráfica de usuario diseñada para simplificar la gestión de contenedores. |
| **Pihole** | *DNS y filtrado* | Contenedor | Aplicación que funciona como un [*sumidero de DNS*](#--sumidero-de-dns) diseñado para proteger tu red frente a contenido no deseado sin necesidad de instalar software en cada cliente. |
| **Nextcloud** | *Cloud y otros* | Contenedor | Plataforma de colaboración de código abierto diseñada para crear tu propia nube privada autoalojada, ofreciendo una alternativa segura a servicios como Google Drive o Dropbox. |
| **Odoo** | *ERP/CRM* | Contenedor | Plataforma de gestión empresarial "todo en uno" de código abierto, diseñada para centralizar todas las operaciones de un negocio en una sola herramienta modular. |
| WIP | WIP | WIP | ***W***ork ***I***n ***P***rogress |

> [!Note]
> #### - Sumidero de DNS 
>   Técnica de seguridad que intercepta consultas DNS de dominios maliciosos o no deseados, devolviendo una IP falsa (generalmente local o "agujero negro") para impedir que los equipos se conecten a sitios dañinos. Funciona como una defensa activa, bloqueando conexiones a botnets o bloqueando publicidad a nivel de red. 

