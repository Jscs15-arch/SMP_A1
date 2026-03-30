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

#### 2.3.1 Instalación de **SSH**

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

#### 2.3.2 Cliente **SSH**
Para conectarnos a nuestro servidor mediante **SSH**, solo debemos tener un cliente de SSH (ya sea gráfico o por CLI) donde agregaremos las credenciales necesarias, el puerto establecido y la IP del servidor (o nombre).

- Mediante cliente GUI ([Terminus](https://termius.com/index.html) en mi caso, ya que funciona tanto en móvil como PC):

> [!Note]
> Se puede usar el cliente deseado, por ejemplo: PuTTY que puede descargase desde su [Web](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) o desde [Microsoft Store](https://apps.microsoft.com/detail/xpfnzksklbp7rj)

- Mediante CLI (Terminal que se desea usar)

```shell
ssh usuario@host_o_ip                   /En caso de puerto predeterminado (22)

ssh -p puerto usuario@host_o_ip         /En caso de puerto personalizado
```

#### 2.3.3 Limpieza de Fingerprint SSH

```shell
ssh-keygen -R [nombre_del_host_o_IP]    /Para limpiar la firgerprint ocupada
```

> [!Note]
> ##### - Fingerprint SSH: 
> Es un resumen corto (hash) de la clave pública de un servidor. Funciona como un sello de identidad único que permite al cliente verificar que se está conectando al servidor correcto y no a un impostor.

## 3. Servicios

### 3.1 Servicios a utilizar

Primero instalaremos los servicios más críticos y necesarios, además de algunos que provee Ubuntu; luego iremos con los referentes a los usuarios y necesidades.

> [!Important]
> No todos son necesarios

| Servicio | Uso | En Sistema/Contenedor | Descripción |
|----------|-----|-----------------------|-------------|
| **NetBird** | *VPN* | Sistema |  Plataforma de código abierto diseñada para crear redes privadas virtuales seguras y de alto rendimiento basadas en el protocolo WireGuard, Una de sus características son sus Redes Mesh Peer-to-Peer que le permite conectar todos los equipos directamente entre sí |
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

### 3.2 Instalación de los servicios

#### 3.2.1 NetBird

NetBird puede ser instalado desde su [página oficial](https://netbird.io/), la cual nos ofrece distintos modos de [instalación](https://app.netbird.io/install). En este caso, lo instalaremos en nuestro PC de escritorio: iniciaremos sesión, lo instalaremos para nuestro sistema operativo correspondiente y accederemos al [panel de control](https://app.netbird.io/peers) de nuestra VPN, aqui podremos administrar y gestionar equipos y usuarios de netbird.

Para agregar el servidor a la red NetBird, solo debemos añadir un *peer* siguiendo los pasos de [instalación para Ubuntu](https://app.netbird.io/install) y al levantar el servicio (`netbird up`) este nos solicitará una Setup Key que crearemos de la siguiente manera:

- Accedemos al panel de Netbird apartado [Setup Key](https://app.netbird.io/setup-keys).

- Crearemos una clave temporal para la instalación (Podemos cambiarlo para 1 día).

- Y utilizaremos esta clave generada para que Netbird se autoconfigure.

Una vez realizado este paso, ya tendríamos el servidor integrado como parte de la red VPN.

> [!Important]
> Recordad que NetBird genera una nueva interfaz de red y, además, actúa como DNS. Por ello, para ciertas configuraciones posteriores, deberemos desactivarlo. Sirva esto para aclarar su funcionamiento con respecto al Firewall.

#### 3.2.2 Uncomplicated FireWall (UFW)

Para UFW tenemos que tener en cuenta los puertos que se desean abrir o cerrar y que usuario (Por IP), red o interfaz queremos que afecte.

> [!Warning]
> Antes que nada, habilitar SSH para mantener el acceso en caso de uso (``sudo ufw allow OpenSSH``)

##### Comandos basicos:

Comprobar actividad:
```bash
sudo ufw status
```

Habilitar UFW:
```bash
sudo ufw allow OpenSSH

sudo ufw enable
```

Ver lista de reglas actuales:
```bash
sudo ufw status verbose

sudo ufw status numbered
```

Desactivar UFW:

```bash
sudo ufw disable
```

##### Bloqueo de direcciones IP y subredes

 Bloquear una IP específica:
```bash
sudo ufw deny from 255.255.255.255 
```

 Bloquear una subred completa:
```bash
sudo ufw deny from 255.255.255.0/24 
```

 Bloquear una IP en una interfaz específica:
```bash
sudo ufw deny in on eth0 from 255.255.255.255
```

##### Permitir direcciones IP:

 Permitir todo el tráfico desde una IP específica:
```bash
sudo ufw allow from 255.255.255.255 /IP
```

 Permitir el tráfico entrante desde una IP en una interfaz específica:
```bash
sudo ufw allow in on eth0 from 255.255.255.255 /IP
```

##### Eliminación de reglas:

 Eliminar una regla por sus parámetros:
```bash
sudo ufw delete allow from 91.198.174.192 
```

 Eliminar una regla por número:
```bash
 sudo ufw status numbered 
 
 sudo ufw delete 1 /Numero del status numbered deseado
```

##### Apertura de puertos comunes
 SSH (puerto 22):
```bash
sudo ufw allow 22 
```

HTTP (puerto 80):
```bash
sudo ufw allow http 
```

HTTPS (puerto 443):
```bash
sudo ufw allow https 
```

HTTP + HTTPS combinado:
```bash
sudo ufw allow proto tcp from any to any port 80,443 
```

##### Permitir una subred:
```bash
sudo ufw allow from 255.255.255.0/24 to any port 3306 
```

> [!Important]
> Se debe tomar precaución al abrir y cerrar puertos ya que podriamos perder acceso o ceder acceso no deseado.

#### Fail2ban

##### Intalación:
```bash
sudo apt update && sudo apt install fail2ban
```

##### Configuración para la protección de SSH (Perzonalizado)

Al instalar Fail2ban este tiene una configuración preestablecida de SSH server la cual puede ser pezonalizada de la siguiente forma (Tambien pueden ser agregados más servicios):

- Accedemos al fichero `sudo nano /etc/fail2ban/jail.conf`

- Buscamos el campo `[sshd]`

- Modificamos con los parametros deseados, [ejemplo](#ejemplo-de-sshd)

- reiniciamos el servicio 
```bash
sudo service fail2ban restart
```

##### Ejemplo de sshd
```bash
[sshd] 

enabled = true 
port    = ssh 
filter  = sshd 
logpath  = /var/log/auth.log 
maxretry = 3 
bantime  = 5m
```
| Variable | Descripción |
|----------|-------------|
| [sshd] | Es el nombre de la sección o "jail" (cárcel). Define las reglas específicas para proteger el servicio de SSH. |
| enabled = true | Indica que esta protección está activada. Si fuera false, Fail2Ban ignoraría estas reglas. |
| port = ssh | Especifica el puerto que debe monitorear. Puede ser el nombre del servicio (ssh) o el número del puerto (por defecto 22). |
| filter = sshd | Le dice a Fail2Ban qué filtro (archivo de configuración con expresiones regulares) debe usar para buscar intentos de acceso fallidos en los registros. |
| logpath = /var/log/auth.log | Es la ruta del archivo de registro donde el sistema guarda los intentos de inicio de sesión. Fail2Ban lo "lee" en tiempo real. |
| maxretry = 3 | Es el número de intentos fallidos permitidos antes de aplicar un bloqueo. |
| bantime = 5m | Es la duración del bloqueo. En este caso, el atacante no podrá intentar conectarse durante 5 minutos. |

##### Comprobación 

1. Realizar intentos de acceso fallidos

2. Comprobamos las IP's enjauladas

- Comando de inspección:
```bash
sudo fail2ban-client status
```

- Inspección del fichero:
```bash
sudo cat /var/log/fail2ban.log | grep Ban
```