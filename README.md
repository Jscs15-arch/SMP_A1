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
| **S.O.** | Ubuntu Server 24.04.4. |

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

#### 2.3.1. Instalación de **SSH**

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
     Loaded: loaded (/usr/lib/systemd/system/ssh.service; disabled; preset: enabled)    # Como vemos el preset se encuentra enabled
          Active: active (running) since Tue 2026-02-24 18:02:45 CET; 16h ago           # Nos centramos en esta linea
          TriggeredBy: ● ssh.socket
```

Si este servicio aparece como desactivado, faltaría activarlo; y si quieres que siempre esté activo al arrancar, utiliza enable.

```bash
sudo systemctl start ssh
sudo system enable ssh
```

#### 2.3.2. Cliente **SSH**
Para conectarnos a nuestro servidor mediante **SSH**, solo debemos tener un cliente de SSH (ya sea gráfico o por CLI) donde agregaremos las credenciales necesarias, el puerto establecido y la IP del servidor (o nombre).

- Mediante cliente GUI ([Terminus](https://termius.com/index.html) en mi caso, ya que funciona tanto en móvil como PC):

> [!Note]
> Se puede usar el cliente deseado, por ejemplo: PuTTY que puede descargase desde su [Web](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) o desde [Microsoft Store](https://apps.microsoft.com/detail/xpfnzksklbp7rj)

- Mediante CLI (Terminal que se desea usar)

```shell
ssh usuario@host_o_ip       #En caso de puerto predeterminado (22)

ssh -p puerto usuario@host_o_ip         #En caso de puerto personalizado
```

#### 2.3.3. Limpieza de Fingerprint SSH

```shell
ssh-keygen -R [nombre_del_host_o_IP]    #Para limpiar la firgerprint ocupada
```

> [!Note]
> ##### - Fingerprint SSH: 
> Es un resumen corto (hash) de la clave pública de un servidor. Funciona como un sello de identidad único que permite al cliente verificar que se está conectando al servidor correcto y no a un impostor.

## 3. Servicios

### 3.1. Servicios a utilizar

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
| **Zammad** | *Ticketing* | Contenedor | Plataforma de gestión de atención al cliente y soporte técnico de código abierto, diseñada para actuar como un centro de mando unificado donde convergen todas las comunicaciones de una organización. |
| **Duplicati** | *Backups* (Copias de seguridad) | Contenedor | Solución de software libre dedicada a la protección de datos mediante la gestión de copias de seguridad cifradas en la nube y sistemas de almacenamiento remoto. |
| WIP | WIP | WIP | ***W***ork ***I***n ***P***rogress |

> [!Note]
> #### - Sumidero de DNS 
>   Técnica de seguridad que intercepta consultas DNS de dominios maliciosos o no deseados, devolviendo una IP falsa (generalmente local o "agujero negro") para impedir que los equipos se conecten a sitios dañinos. Funciona como una defensa activa, bloqueando conexiones a botnets o bloqueando publicidad a nivel de red. 

### 3.2. Instalación de los servicios

#### 3.2.1. NetBird

NetBird puede ser instalado desde su [página oficial](https://netbird.io/), la cual nos ofrece distintos modos de [instalación](https://app.netbird.io/install). En este caso, lo instalaremos en nuestro PC de escritorio: iniciaremos sesión, lo instalaremos para nuestro sistema operativo correspondiente y accederemos al [panel de control](https://app.netbird.io/peers) de nuestra VPN, aqui podremos administrar y gestionar equipos y usuarios de netbird.

Para agregar el servidor a la red NetBird, solo debemos añadir un *peer* siguiendo los pasos de [instalación para Ubuntu](https://app.netbird.io/install) y al levantar el servicio (`netbird up`) este nos solicitará una Setup Key que crearemos de la siguiente manera:

- Accedemos al panel de Netbird apartado [Setup Key](https://app.netbird.io/setup-keys).

- Crearemos una clave temporal para la instalación (Podemos cambiarlo para 1 día).

- Y utilizaremos esta clave generada para que Netbird se autoconfigure.

Una vez realizado este paso, ya tendríamos el servidor integrado como parte de la red VPN.

> [!Important]
> Recordad que NetBird genera una nueva interfaz de red y, además, actúa como DNS. Por ello, para ciertas configuraciones posteriores, deberemos desactivarlo. Sirva esto para aclarar su funcionamiento con respecto al Firewall y DNS.

> [!Warning]
> [**OBSOLETO**]
>
> Después de la versión [**0.70.5**] de NetBird este ignora script y otros porque le han agregado nueva gestión de contenido incluyendo el DNS.
>
> [Proceso obligatorio para deshabilitar DNS de NetBird](#deshabilitar-dns-de-netbird-en-panel)
>
> ##### Script para prevención de bloqueo de DNS
> ```bash
> #!/bin/bash
> 
> echo "Arrancando daemon de NetBird..."
> echo "Esperando a que exista el socket..."
> while [ ! -S /var/run/netbird.sock ]; do
>     sleep 1
> done
> echo "Esperando a Pi-hole..."
> while ! docker ps | grep -q pihole; do
>         sleep 2
> done
> echo "Esperando a que Pi-hole use el puerto 53..."
> # Espera hasta que el puerto 53 esté ocupado por docker/pihole
> while ! ss -tulpn | grep -q ":53"; do
>      sleep 2
> done
> echo "Puerto 53 activo, arrancando NetBird sin DNS..."
> # Arrancar NetBird sin DNS
> netbird up --disable-dns
> ```
>
> ##### Implementar script en arranque
> 1. Crear fichero para el daemon [/etc/systemd/system/script_daemon](#daemon)
> 2. Ejecutar sudo systemctl daemon-reload.
> 3. Ejecutaste sudo systemctl enable --now script.
> 
> ###### **Daemon**
> ```bash
> [Unit]
> Description=Start NetBird after Pi-hole
> After=docker.service network-online.target
> Wants=network-online.target
>
> [Service]
> Type=simple
> ExecStart=/usr/local/bin/script.sh
> Restart=on-failure
>
> [Install]
> WantedBy=multi-user.target
> ```

##### **Deshabilitar DNS de NetBird en panel**

1. Acceder al panel de NetBird en su [apartado DNS](https://app.netbird.io/dns/settings)

2. Agregar a un grupo al que no se desee que tenga DNS

3. Finalizamos añadiendo el grupo que restringe DNS a los [peers](https://app.netbird.io/peers) necesarios (En este caso el servidor, ya que usaremos un DNS personalizado).

#### 3.2.2. Uncomplicated FireWall (UFW)

Para UFW tenemos que tener en cuenta los puertos que se desean abrir o cerrar y que usuario (Por IP), red o interfaz queremos que afecte.

##### Instalación
UFW
```bash
sudo apt update && sudo apt install ufw -y
```
UFW-docker 
Script para cambiar reglas de docker
```bash
# 1. Descargar el script oficial
sudo wget -O /usr/local/bin/ufw-docker https://github.com/chaifeng/ufw-docker/raw/refs/heads/master/ufw-docker

# 2. Dar permisos de ejecución
sudo chmod +x /usr/local/bin/ufw-docker

# 3. Modificar la configuración de UFW para que interactúe con Docker
# Este comando añade las reglas necesarias a /etc/ufw/after.rules
sudo ufw-docker install
```

> [!Warning]
> Antes que nada, habilitar SSH para mantener el acceso en caso de uso (``sudo ufw allow OpenSSH``)

> [!Note]
> Todos los comandos pueden ser usados para ufw-docker agregando `ufw-docker` al comando

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
sudo ufw delete allow from 255.255.255.255
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
sudo ufw allow from 255.255.255.0/24 to any port 22
```

> [!Important]
> Se debe tomar precaución al abrir y cerrar puertos ya que podriamos perder acceso o ceder acceso no deseado.

#### 3.2.3. Fail2ban

##### Intalación:
```bash
sudo apt update && sudo apt install fail2ban
```

##### Configuración para la protección de SSH (Perzonalizado)

Al instalar Fail2ban este tiene una configuración preestablecida de SSH server la cual puede ser pezonalizada de la siguiente forma (Tambien pueden ser agregados más servicios):

1. Accedemos al fichero `sudo nano /etc/fail2ban/jail.conf`

2. Buscamos el campo `[sshd]`

3. Modificamos con los parametros deseados, [ejemplo](#ejemplo-de-sshd)

4. reiniciamos el servicio 
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
| [sshd] | Es el nombre de la sección o jail. Define las reglas específicas para proteger el servicio de SSH. |
| enabled = true | Indica que esta protección está activada. Si fuera false, Fail2Ban ignoraría estas reglas. |
| port = ssh | Especifica el puerto que debe monitorear. Puede ser el nombre del servicio o el número del puerto (ssh por defecto 22). |
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

#### 3.2.4. Apache2

> [!Important]
> Apache2 se utilizará como **reverse proxy**. En la sección de cada servicio, detallaré cómo agregarlo y los requisitos específicos necesarios.
> 
> En este apartado solo encontrarás explicaciones sobre las herramientas que utilizaremos y enlaces a los archivos de configuración correspondientes.

##### Instalación

```bash
sudo apt update $$ sudo apt install apache2 -y
```


> [!Note]
> Al finalizar la descarga podemos pasar a comprobar el estado y en caso de encontrase deshabilitado habilitarlo.

- **Estado**
```bash
sudo systemctl status apache2
```
- **habilitar**
```bash
sudo systemctl start apache2
```
- **habilitar al arrancar**
```bash
sudo systemctl enable apache2
```

###### **Extenciones**

| Modulo | Comando para activar | Uso principal |
|--------|----------------------|---------------|	
|SSL | sudo a2enmod ssl | Habilitar HTTPS y certificados |
|Headers | sudo a2enmod headers | Modificar cabeceras de respuesta |
|Forward/Proxy | sudo a2enmod proxy | Base para reenviar peticiones a otros servidores |
|Proxy HTTP| sudo a2enmod proxy_http | Necesario para hacer Reverse Proxy a apps |
|Rewrite | sudo a2enmod rewrite | Para reglas de redirección de URLs |

###### 1. Verifica que no haya errores de sintaxis
```bash
sudo apache2ctl configtest
```

###### 2. Si dice "Syntax OK", reinicia Apache
```bash
sudo systemctl restart apache2
```

##### Generar certificado autofirmado
Se deben definir los días 
```bash
sudo openssl req -x509 -nodes -days 3650 -newkey rsa:2048 \
  -keyout /etc/apache2/ssl/apache.key \
  -out /etc/apache2/ssl/apache.crt \
  -subj "/C=ES/CN=*.dominio.com" \
  -addext "subjectAltName = DNS:dominio.com, DNS:*.dominio.com"
```

##### **Virtual host's para reverse proxy**

> [!Important]
> Cambiar puertos a los establecidos en los stacks (compose)

> [!Warning]
> No establecer un puerto a multiples servicios

- [Reenvio del puerto 80 a 443](#reenvio-del-puerto-80-a-443-http-a-https)

- [Pi-hole](#reverse-proxy-para-pi-hole)

- [Nextcloud](#reverse-proxy-para-nextcloud)
    - [Signaling](#reverse-proxy-para-signaling)
    - [Onlyoffice](#reverse-proxy-para-onlyoffice)

- [Webmin](#reverse-proxy-para-webmin)

- [Odoo](#reverse-proxy-para-odoo)

- [Portainer](#reverse-proxy-para-portainer)

- [Zammad](#reverse-proxy-para-zammad)

- [Duplicati](#reverse-proxy-para-duplicati)

###### **Reenvio del puerto 80 a 443 (HTTP a HTTPS):**

```bash
<VirtualHost *:80>
  ServerName dominio.com
  RewriteEngine On
  RewriteCond %{HTTPS} off
  RewriteRule ^ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]
</VirtualHost>
```

#### 3.2.5. Webmin

##### **Instalación**

 - Setup
```bash
curl -o webmin-setup-repo.sh https://raw.githubusercontent.com/webmin/webmin/master/webmin-setup-repo.sh
sudo sh webmin-setup-repo.sh
```

- apt
```bash
sudo apt-get install webmin --install-recommends
```

##### Configuraciones perzonalizadas

Acceder al fichero `/etc/webmin/miniserv.conf` para configurar distintas cosas.

> [!Note]
> Estas configuraciones pueden ser modificadas tambien desde la interfaz de Webmin.
```bash
port=10000
ssl=0    # Esto causa que webmin abra en http pero el proxy se encargara de cambiar de https del cliente a http al servidor
behind_proxy=1
cookie_secure=1
no_ssl2=1
no_ssl3=1
listen=10000
logouttimes=
referers_none=1
trust_real_ip=1
cookiepath=/
redirect_port=443
redirect_ssl=1
relative_redir=1
no_redirect_port=1
host=sub.dominio.com
proxies=127.0.0.1
no_resolv_myname=0
websockets_ssl=1
websockets_port=443
websockets_host=sub.dominio.com
ext_prefix_https=1
xterm_port=443
xterm_protocol=wss
```

> [!Important]
> En caso de mal funcinamiento de websockets configurar en Webmin lo siguiente

###### **Configuración de websockets en Webmin**

**Ruta:** Panel izquierdo de Webmin → Webmin → Configuración de Webmin → Puertos y Direcciones → Nombre de host para conexiones WebSockets

Establecer como manual y agregar `wss://sub.dominio.com` y Nombre de servidor Web `sub.dominio.com`

##### **Reverse proxy para Webmin**
```bash
<VirtualHost *:443>
  ServerName sub.dominio.com

  # Establecer certificado para https
  SSLEngine on
  SSLCertificateFile /etc/apache2/ssl/certificado.crt
  SSLCertificateKeyFile /etc/apache2/ssl/clavepriv.key

  # Websocket http (Cambiado a ws:// porque Webmin tiene ssl=0)
  RewriteEngine on
  RewriteCond %{HTTP:Upgrade} websocket [NC]
  RewriteCond %{HTTP:Connection} upgrade [NC]
  RewriteRule ^/?(.*) "ws://127.0.0.1:10000/$1" [P,L]
    
  # http porque Webmin tiene ssl=0 y el puerto en que se encuentra
  ProxyPreserveHost ON
  ProxyPass / http://127.0.0.1:10000/
  ProxyPassReverse / http://127.0.0.1:10000/
    
  # CABECERAS (Importantes para que Webmin sepa que el usuario usa HTTPS)
  RequestHeader set X-Forwarded-Proto "https"
  RequestHeader set X-Forwarded-Port "443"
  ProxyPassReverseCookiePath / /
</VirtualHost>
```

#### 3.2.6. Docker

1. Eliminar versiones antiguas de Docker Engine
      ```bash      
      for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
      ```
2. : Incluir el repositorio de Docker CE

      ```bash
        # Add Docker's official GPG key:
        sudo apt-get update
        sudo apt-get install ca-certificates curl
        sudo install -m 0755 -d /etc/apt/keyrings
        sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
        sudo chmod a+r /etc/apt/keyrings/docker.asc
        
        # Add the repository to Apt sources:
        echo \
        "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc]https://download.docker.com/linux/ubuntu \
        $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
        sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
        sudo apt-get update
      ```
3. Instalar Docker Engine CE

    Actualizar el índice de paquetes e instalar la última versión de Docker Engine CE
      ```bash
      sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
      ```
4. Comprobar la instalación
    ```bash
    sudo docker version
    ```

> [!Note]
> **Recomendación post instalación**
>
> ##### **Permitir administrar Docker con usuarios sin privilegios**
>
> - Crear el grupo docker
> ```bash
>   sudo groupadd docker
> ```
>
> - Añadir a los usuarios deseados a ese grupo
> ```bash
>   sudo usermod -aG docker $USER
> ```
>
> - Especificar que el grupo docker administra el fichero docker.sock
> ```bash
>   newgrp docker
> ```
>
> - Comprobación
> ```bash
>   docker run hello-world
> ```

> [!Note]
> Todos los stacks (Compose) pueden ser usados tanto 

> [!Important]
> Para todos los contenedores se debe tener en cuenta los **puertos ocupados**

#### 3.2.6. Portainer

##### **Instalación**

```bash
docker run -d \
--name portainer \
--restart=always \
-p 127.0.0.1:8000:8000 \
-p 127.0.0.1:9443:9443 \
-v /var/run/docker.sock:/var/run/docker.sock \
-v /ruta:/data \
portainer/portainer-ce:lts
```

> [!Note]
> - **docker run**
> Inicia un nuevo contenedor Docker a partir de una imagen.
> 
> - **-d**
> Ejecuta el contenedor en segundo plano (detached mode).
> 
> - **-p "127.0.0.1:8000:8000"**
> Mapea el puerto 8000 del host al puerto 8000 del contenedor.
> 127.0.0.1 → solo accesible localmente
> Primer 8000 → puerto del host
> Segundo 8000 → puerto interno del contenedor
>
> - **-p "127.0.0.1:9443:9443"**
> Mapea el puerto HTTPS principal de Portainer.
> Esto permite acceder a la interfaz web mediante:
> https://localhost:9443 o solo pueda acceder el servidor local y el reverse proxy  
> 
> - **--name portainer**
> Asigna el nombre portainer al contenedor.
> 
> - **--restart=always**
> Hace que el contenedor se reinicie automáticamente:
> si falla
> si Docker se reinicia
> si el sistema operativo se reinicia
> - **-v /var/run/docker.sock:/var/run/docker.sock**
> Monta el socket de Docker dentro del contenedor.
> Esto permite que Portainer administre Docker del host.
> 
> - **-v /ruta:/data**
> Crea y monta un volumen persistente donde lo establescas.
> Ahí se guardan:
> configuraciones
> usuarios
> contraseñas
> datos de Portainer
>
> - **portainer/portainer-ce:lts**
> Imagen Docker que se ejecutará:
> portainer/portainer-ce: imagen oficial Community Edition
> lts: versión Long Term Support

##### **Reverse proxy para Portainer**
```bash
<VirtualHost *:443>
    ServerName sub.dominio.com

    # Establecer certificado para https
    SSLEngine on
    SSLCertificateFile /etc/apache2/ssl/certificado.crt
    SSLCertificateKeyFile /etc/apache2/ssl/clavepriv.key

    # Soporte para WebSockets (Consola de Contenedores y datos)
    RewriteEngine on
    RewriteCond %{HTTP:Upgrade} websocket [NC]
    RewriteCond %{HTTP:Connection} upgrade [NC]
    RewriteRule ^/?(.*) "wss://127.0.0.1:9443/$1" [P,L]

    ProxyPreserveHost On
    ProxyPass / https://127.0.0.1:9443/
    ProxyPassReverse / https://127.0.0.1:9443/

    # Deshabilitar verificación solo para SSL autofirmado
    SSLProxyEngine on
    SSLProxyVerify none
    SSLProxyCheckPeerCN off
    SSLProxyCheckPeerName off
</VirtualHost>
```

#### 3.2.7. Pi-hole

##### **Instalación mediante archivo Docker compose (Stack)**

```yaml
services:
  pihole:
    image: pihole/pihole:latest
    container_name: pihole
    restart: unless-stopped
    networks:
      - "nombre_de_red"
    ports:
      # UI limitada a reverse proxy (IP del reverse)
      - "127.0.0.1:8080:80" # Cambiar a puerto disponible
      # Deben estar abiertos para proveer DNS
      - "IP:53:53/tcp"
      - "IP:53:53/udp"
    environment:
      TZ: "Europe/Madrid"
      WEBPASSWORD: "Clave_para_UI"
    dns:
      - 127.0.0.1
    volumes:
      - ./etc:/etc/pihole
      - ./dnsmasq:/etc/dnsmasq.d
      # Certificado para autenticar uso de Iframe
      - /etc/apache2/ssl/certificado.crt:/usr/local/share/ca-certificates/certificado.crt:ro
    extra_hosts:
      - "sub.dominio.com:IP" # servidor apache
    # Actualizar certificados
    command: >
      /bin/sh -c "update-ca-certificates && /entrypoint.sh"

networks:
  nombre_de_red:
    driver: bridge
    name: nombre_de_red
```

##### **Configuraciones**

###### **Interfaz a la cual respondera DNS**

**Ruta:** Panel izquierdo de Pi-hole → Settings → DNS → Interface settings

Al ser un contenedor por defecto estara resolviendo solo para la red docker por ello debemos cambiar la interfaz donde respondera a la que usa el contenedor para comunicarse con el servidor

> [!Warning]
> Asegurar la correcta implementacíon de firewall para DNS (Puerto 53) 

###### **Local DNS**

**Ruta:** Panel izquierdo de Pi-hole → Settings → Local DNS Records

Agregamos IP's a **`List of local DNS records`**

###### **Servidores upstream**

Son DNS externos a los que Pi-hole envía las consultas legítimas después de bloquear los dominios no deseados.

**Ruta:** Panel izquierdo de Pi-hole → Settings → DNS → Upstream DNS Servers

Agregamos DNS's de Google u otros proveedores, en caso de usar unbound debera ser agregado como `IP#5335`

##### **Reverse proxy para Pi-hole**
```bash
<VirtualHost *:443>
  ServerName sub.dominio.com

  # Proxy HTTP
  ProxyPreserveHost On
  ProxyPass / http://127.0.0.1:8080/
  ProxyPassReverse / http://127.0.0.1:8080/

  # WebSocket
  RequestHeader set X-Forwarded-Proto "https"
  RequestHeader set X-Forwarded-Ssl "on"

  # PERMITIR IFRAME
  Header always unset X-Frame-Options
  Header always set Content-Security-Policy "frame-ancestors 'self' https://sub.dominio.com" # debe ser el dominio que accedera mediante Iframe

  ProxyTimeout 300

  SSLEngine on
  SSLCertificateFile /etc/apache2/ssl/certificado.crt
  SSLCertificateKeyFile /etc/apache2/ssl/clavepriv.key
</VirtualHost>
```

#### Nextcloud

##### **Instalación mediante archivo Docker compose (Stack)**

```yaml
services:
  db:
    image: mariadb:10.11
    container_name: nextcloud-db
    restart: unless-stopped
    command:
      - --transaction-isolation=READ-COMMITTED
      - --binlog-format=ROW
      - --innodb_buffer_pool_size=2G
      - --innodb_log_file_size=256M
      - --wait_timeout=28800
      - --interactive_timeout=28800
      - --max_connections=200
    volumes:
      - ./cloud/db:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=clave_root
      - MYSQL_PASSWORD=clave_DB
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud

  cache:
    image: redis:alpine
    container_name: nextcloud-redis
    restart: always

  app:
    image: nextcloud:latest
    container_name: nextcloud-app
    restart: unless-stopped
    ports:
      - "127.0.0.1:8080:80"
    depends_on:
      - db
      - cache
    # dominios que deseas que conozca el contenedor
    extra_hosts:
      - "sub.dominio.com:192.168.1.100"
      - "sub.dominio.com:192.168.1.100"
      - "sub.dominio.com:192.168.1.100"
      - "sub.dominio.com:192.168.1.100"
    volumes:
      - ./cloud/data:/var/www/html
      - /etc/apache2/ssl/certificado.crt:/usr/local/share/ca-certificates/certificado.crt:ro
    environment:
      - MYSQL_PASSWORD=clave_DB
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - MYSQL_HOST=db
      - REDIS_HOST=cache
      - REDIS_HOST_PORT=6379
      - MEMCACHE_LOCKING=\OC\Memcache\Redis
      - MEMCACHE_LOCAL=\OC\Memcache\APCu
      - PHP_MEMORY_LIMIT=1G
      - PHP_UPLOAD_LIMIT=10G
    command: /bin/sh -c "update-ca-certificates && apache2-foreground"

  nats:
    image: nats:2.10-alpine
    container_name: nextcloud-nats
    restart: unless-stopped

  signaling:
    image: strukturag/nextcloud-spreed-signaling:latest
    container_name: nextcloud-signaling
    restart: unless-stopped
    ports:
      - "127.0.0.1:8080:8080"
    depends_on:
      - nats
    environment:
      - LISTEN=0.0.0.0:8080
      - GNATS_URL=nats://nats:4222
      # Dominio nextcloud
      - NC_DOMAIN=sub.dominio.com
      - NC_SECRET=Clave
      - SESSIONS_HASHKEY=key
      - SESSIONS_BLOCKKEY=key
    volumes:
      - ./cloud/signaling/config:/config
      - /etc/apache2/ssl/certificado.crt:/usr/local/share/ca-certificates/certificado.crt:ro
    extra_hosts:
      - "sub.dominio.com:192.168.1.100"
    command: >
      /bin/sh -c "update-ca-certificates && /entrypoint.sh"

  onlyoffice:
    image: onlyoffice/documentserver
    container_name: nextcloud-onlyoffice
    ports:
      - "127.0.0.1:9981:80"
    volumes:
      - /etc/apache2/ssl/certificado.crt:/usr/local/share/ca-certificates/certificado.crt:ro
    environment:
      - JWT_ENABLED=true
      - JWT_SECRET=clave_JWT
      - USE_UNAUTHORIZED_STORAGE=true
    restart: always
    command: >
      bash -c "update-ca-certificates"
    extra_hosts:
      - "sub.dominio.com:192.168.1.100"
```

##### **Reverse proxy para Nextcloud**
```bash
<VirtualHost *:443>
    ServerName sub.dominio.com

    SSLEngine on
    SSLCertificateFile /etc/apache2/ssl/certificado.crt
    SSLCertificateKeyFile /etc/apache2/ssl/clavepriv.key

    ProxyPreserveHost On
    AllowEncodedSlashes NoDecode

    ProxyPass / http://127.0.0.1:8080/ nocanon
    ProxyPassReverse / http://127.0.0.1:8080/

    RequestHeader set X-Forwarded-Proto "https"
    RequestHeader set X-Forwarded-Host "sub.dominio.com"
    RequestHeader set X-Forwarded-For %{REMOTE_ADDR}s

    # CSP para Iframe
    Header unset Content-Security-Policy
    Header always set Content-Security-Policy "script-src 'self' sub.domini.com sub.dominio.com;"

    SetEnvIf Authorization "(.*)" HTTP_AUTHORIZATION=$1
</VirtualHost>
```

##### Servicios externos e integraciones

###### **Implementación de correo Gmail**

- Para ello debemos seguir el [manual](https://support.google.com/accounts/answer/185833?dark=1&hl=es-419) de google y añadir una (clave de aplicación)[https://myaccount.google.com/u/1/apppasswords]

- Ahora podemos acceder al correo con esta clave de aplicación y el Gmail con el que se creo la clave

- Agregar los servidores
  - imap: imap.gmail.com
  - smtp: smtp.gmail.com

- Ahora solo guardamos y se conectara

###### **Implementación de notificaciones Zammad**

1. Acceder al usuario zammad 

2. Crear el (token)[https://soporte.server.home/#profile/token_access]

3. Acceder desde nextcloud con el token

###### **Implementación de Iframe**

- Lo primero a tener en cuenta son los CSP, ya que estos pueden bloquear las solicitudes (Esto fue definido en los virtualhost de los servicios)

1. Instalar iFrame Widget en aplicaciones de nextcloud

2. Ahora tenemos dos paneles de conexiones Iframe, uno para el usuario y otro del sistema que solo puede cambiar el administrador

  - Iframe widget desde panel administrador

    Podemos crear una conexión agregando el dominio deseado y si es publico o de un grupo especifico

  - Iframe widget desde panel usuario

    Podemos crear o quitar Iframes solo para el usuario

###### **Implementación de Onlyoffice**

Para conectar onlyoffice como dominio necesitaremos que el contenedor conozca el dominio o el DNS

###### **reverse proxy para onlyoffice**
```bash
<VirtualHost *:443>
    ServerName sub.dominio.com

    SSLEngine on
    SSLCertificateFile /etc/apache2/ssl/certificado.crt
    SSLCertificateKeyFile /etc/apache2/ssl/clavepriv.key

    ProxyPreserveHost On
    AllowEncodedSlashes NoDecode

    # PROXY PRINCIPAL
    ProxyPass / http://127.0.0.1:9981/ nocanon
    ProxyPassReverse / http://127.0.0.1:9981/

    # CABECERAS
    RequestHeader set X-Forwarded-Proto "https"
    RequestHeader set X-Forwarded-Host "sub.dominio.com"
    RequestHeader set X-Forwarded-Port "443"
    RequestHeader set X-Forwarded-Ssl "on"
    RequestHeader add X-Forwarded-For %{REMOTE_ADDR}s

    # WEBSOCKETS
    ProxyPassMatch "^/(.*)/websocket"  ws://127.0.0.1:9981/$1/websocket
    ProxyPass "/websocket"  ws://127.0.0.1:9981/websocket

    # Mantener conexión
    ProxyTimeout 600

</VirtualHost>
```

Una vez hecho lo anterior procedemos a agregar el servidor onlyoffice

**RUTA: Configuraciones de administración → ONLYOFFICE → Ajustes de servidor**

- Dirección de ONLYOFFICE Docs
  
  [dominio establecido](#L1006)

- Clave secreta

  [clave establcida](#L925)

- Ajustes de servidor avanzados 

  - Encabezado de autenticación (dejar en blanco para utilizar el encabezado predeterminado)
    
    **Authorization**

  - Dirección de ONLYOFFICE Docs para solicitudes internas del servidor
    
    **https://sub.dominio.com/**


###### **Motor de alto rendimiento (Signaling)**

Para conectar signaling como dominio necesitaremos que el contenedor conozca el dominio o el DNS

###### **reverse proxy para signaling**
```bash
<VirtualHost *:443>
    ServerName sub.dominio.com

    SSLEngine on
    SSLCertificateFile /etc/apache2/ssl/certificado.crt
    SSLCertificateKeyFile /etc/apache2/ssl/clavepriv.key

    ProxyPreserveHost On

    RequestHeader set X-Forwarded-Proto "https"

    # WebSockets global
    RewriteEngine On
    RewriteCond %{HTTP:Upgrade} =websocket [NC]
    RewriteRule /(.*) ws://127.0.0.1:8100/$1 [P,L]

    # HTTP normal
    ProxyPass / http://127.0.0.1:8100/
    ProxyPassReverse / http://127.0.0.1:8100/

    # Headers websockets
    ProxyPassReverseCookieDomain 127.0.0.1 sub.dominio.com
</VirtualHost>
```

Una vez hecho lo anterior procedemos a agregar el motor de alto rendimiento para nextcloud talk

**RUTA: Configuraciones de administración → Talk → Motor de alto rendimiento**

- Colocar el [dominio establecido](#L1042)

- Colocar la [clave establecida](#L925)

> [!Note]
> Como hemos establecido el certificado en el contenedor podemos validar certificado SSL.

#### Odoo

##### **Instalación mediante archivo Docker compose (Stack)**

```yaml
services:
#Definimos el servicio Web, en este caso Odoo
  web:
    #Indicamos que imagen de Docker Hub utilizaremos
    image: odoo:18
    container_name: odoo-web
    restart: unless-stopped
    #Indicamos que depende de "db", por lo cual debe ser procesada>
    depends_on:
        - db

    # Port Mapping: indicamos que el puerto 8069 del contenedor se>
    # Permitiendo acceder a Odoo mediante http://localhost:8069
    ports:
      - "127.0.0.1:8069:8069"
      - "127.0.0.1:8072:8072"

    # Mapeamos el directorio de los contenedores (como por ejemplo>
    # en un directorio local (como por ejemplo en un directorio ".>
    # situado en el lugar donde ejecutemos "Docker compose"
    volumes:
      - ./volumesOdoo/addons:/mnt/extra-addons
      - ./volumesOdoo/odoo-web-data:/var/lib/odoo
      - ./volumesOdoo/config:/etc/odoo
    #Indicamos que el contenedor funcionara con usuario root y no >
    user: root
    # Definimos variables de entorno de Odoo
    environment:
      - DB_HOST=db
      - DB_PORT=5432
      - DB_USER=odoo
      - DB_PASSWORD=clave
      - DB_NAME=odoo_db
#Definimos el servicio de la base de datos
  db:
    image: postgres:15
    container_name: odoo-db
    restart: unless-stopped
    # Definimos variables de entorno de PostgreSQL
    environment:
      - POSTGRES_PASSWORD=clave
      - POSTGRES_USER=odoo
      - POSTGRES_DB=postgres
    # Mapeamos el directorio del contenedor "var/lib/postgresql/da>
    # situado en el lugar donde ejecutemos "Docker compose"
    volumes:
      - ./volumesOdoo/dataPostgreSQL:/var/lib/postgresql/data
```

##### **Reverse proxy para Odoo**
```bash
<VirtualHost *:443>
    ServerName sub.dominio.com

    ProxyPreserveHost On
    ProxyAddHeaders On
    RequestHeader set X-Forwarded-Proto "https"
    RequestHeader set X-Forwarded-Ssl "on"

    # Error 400)
    RequestHeader set X-Forwarded-Host %{HTTP_HOST}e

    # Tráfico de tiempo real (Websocket http)
    ProxyPass /websocket ws://127.0.0.1:8072/websocket
    ProxyPassReverse /websocket ws://127.0.0.1:8072/websocket

    # Tráfico de Longpolling
    ProxyPass /longpolling http://127.0.0.1
    ProxyPassReverse /longpolling http://127.0.0.1

    # Tráfico (Main)
    ProxyPass / http://127.0.0.1:8069/
    ProxyPassReverse / http://127.0.0.1:8069/

    SSLEngine on
    SSLCertificateFile /etc/apache2/ssl/certificado.crt
    SSLCertificateKeyFile /etc/apache2/ssl/calvepriv.key

    # Tiempo de espera para evitar cortes en el chat
    ProxyTimeout 720
</VirtualHost>
```

#### Zammad

##### **Instalación mediante archivo Docker compose (Stack)**

```yaml
x-shared:
  app: &app
    environment: &env
      MEMCACHE_SERVERS: memcached:11211
      POSTGRESQL_DB: zammad
      POSTGRESQL_HOST: postgresql
      POSTGRESQL_USER: zammad
      POSTGRESQL_PASS: clave
      POSTGRESQL_PORT: 5432
      POSTGRESQL_OPTIONS: ?pool=50

      ZAMMAD_FQDN: sub.dominio.com
      NGINX_SERVER_NAME: sub.dominio.com

      REDIS_URL: redis://redis:6379

      ELASTICSEARCH_ENABLED: "true"
      ELASTICSEARCH_HOST: elasticsearch
      ELASTICSEARCH_PORT: 9200

      BACKUP_DIR: "/var/tmp/zammad"
      BACKUP_TIME: "03:00"
      HOLD_DAYS: "10"
      TZ: "Europe/Madrid"

    image: ghcr.io/zammad/zammad:7.0.1-0024
    restart: always
    volumes:
      - ./data/backup:/var/tmp/zammad:ro
      - ./data/storage:/opt/zammad/storage
    depends_on:
      - memcached
      - postgresql
      - redis

services:
  backup:
    <<: *app
    command: ["zammad-backup"]
    container_name: zammad-backup
    volumes:
      - ./data/backup:/var/tmp/zammad
      - ./data/storage:/opt/zammad/storage
    user: 0:0

  elasticsearch:
    image: elasticsearch:8.11.0
    container_name: zammad-elasticsearch
    restart: always
    volumes:
      - ./data/elasticsearch:/usr/share/elasticsearch/data
    user: "1000:1000"
    environment:
      discovery.type: single-node
      xpack.security.enabled: 'false'
      ES_JAVA_OPTS: -Xms512m -Xmx512m
      TZ: "Europe/Madrid"
    mem_limit: 1g
    cpus: 1.0

  init:
    <<: *app
    command: ["zammad-init"]
    container_name: zammad-init
    depends_on:
      - postgresql
    restart: on-failure
    user: 0:0

  memcached:
    image: memcached:1.6.41-alpine
    container_name: zammad-memcached
    command: memcached -m 256M
    restart: always

  nginx:
    <<: *app
    command: ["zammad-nginx"]
    container_name: zammad-nginx
    ports:
      - "127.0.0.1:8080:8080"
    depends_on:
      - railsserver

  postgresql:
    image: postgres:17.9-alpine
    container_name: zammad-postgresql
    restart: always
    environment:
      POSTGRES_DB: zammad
      POSTGRES_USER: zammad
      POSTGRES_PASSWORD: clave
      TZ: "Europe/Madrid"
    volumes:
      - ./data/postgresql:/var/lib/postgresql/data

  railsserver:
    <<: *app
    command: ["zammad-railsserver"]
    container_name: zammad-railsserver
    networks:
      default:
        aliases:
          - zammad-railsserver

  redis:
    image: redis:8.6.2-alpine
    container_name: zammad-redis
    restart: always
    volumes:
      - ./data/redis:/data

  scheduler:
    <<: *app
    command: ["zammad-scheduler"]
    container_name: zammad-scheduler

  websocket:
    <<: *app
    command: ["zammad-websocket"]
    container_name: zammad-websocket
```

##### **Reverse proxy para Zammad**
```bash
<VirtualHost *:443>
    ServerName sub.dominio.com

    SSLEngine on
    SSLCertificateFile /etc/apache2/ssl/certificado.crt
    SSLCertificateKeyFile /etc/apache2/ssl/clavepriv.key

    ProxyPreserveHost On
    ProxyRequests Off

    # Cabeceras importantes
    RequestHeader set X-Forwarded-Proto "https"
    RequestHeader set X-Forwarded-SSL "on"

    # Proxy HTTP
    ProxyPass / http://127.0.0.1:8080/
    ProxyPassReverse / http://127.0.0.1:8080/

    # WebSocket (Zammad usa /cable)
    ProxyPass /cable ws://127.0.0.1:8080/cable
    ProxyPassReverse /cable ws://127.0.0.1:8080/cable

    # PERMITIR IFRAME
    Header always unset X-Frame-Options
    Header always set Content-Security-Policy "frame-ancestors 'self' https://sub.dominio.com" # debe ser el dominio que accedera mediante Iframe

    LimitRequestBody 104857600
    ProxyTimeout 300

    # Logs
    ErrorLog ${APACHE_LOG_DIR}/zammad_error.log
    CustomLog ${APACHE_LOG_DIR}/zammad_access.log combined
</VirtualHost>
```

#### Duplicati

##### **Instalación mediante archivo Docker compose (Stack)**

```yaml
services:
  duplicati:
    image: lscr.io/linuxserver/duplicati:latest
    container_name: duplicati
    environment:
      - PUID=0
      - PGID=0
      - TZ=Europe/Madrid
      - DUPLICATI__WEBSERVICE_PASSWORD=clave
      - DUPLICATI__WEBSERVICE_ALLOWED_HOSTNAMES=sub.dominio.com
      - SETTINGS_ENCRYPTION_KEY=key
      - UMASK=002
      - DUPLICATI__WEBSERVICE_INTERFACE=any
      - DUPLICATI__WEBSERVICE_EXTERNAL=true
    volumes:
      - ./config:/config
      - /etc/apache2/ssl/certificado.crt:/usr/local/share/ca-certi>
      # Destino local para las copias
      - /ruta/backups:/backups
      # Destino remoto o USB para las copias
      - /mnt/backups:/backups_remoto
      # Destinos a copiar
      - /ruta/docker:/source/docker
      - /etc:/source/sistema
      - /home:/source/usuarios
      - /var/www:/source/a2webs
      # Destino para workbench
      - /ruta/tmp_works:/temp_duplicati

    ports:
      - "127.0.0.1:8200:8200"
    restart: unless-stopped
```

##### **Reverse proxy para Duplicati**
```bash
<VirtualHost *:443>
    ServerName sub.dominio.com

    SSLEngine on
    SSLCertificateFile /etc/apache2/ssl/certificado.crt
    SSLCertificateKeyFile /etc/apache2/ssl/clavepriv.key

    # Soporte WebSockets
    RewriteEngine on
    RewriteCond %{HTTP:Upgrade} websocket [NC]
    RewriteCond %{HTTP:Connection} upgrade [NC]
    RewriteRule ^/?(.*) "ws://127.0.0.1:8200/$1" [P,L]

    ProxyPreserveHost On

    # Cookies
    ProxyPassReverseCookiePath / /
    # Forzamos SameSite=None para que la sesión no se pierda en el>
    Header edit Set-Cookie ^(.*)$ "$1; HttpOnly; Secure; SameSite=>

    # Proxy
    ProxyPass / http://127.0.0.1:8200/
    ProxyPassReverse / http://127.0.0.1:8200/

    # Cabeceras de confianza
    RequestHeader set X-Forwarded-Proto "https"
    RequestHeader set X-Forwarded-Port "443"
    RequestHeader set X-Forwarded-Host "sub.dominio.com"

    ProxyTimeout 600

    ErrorLog ${APACHE_LOG_DIR}/duplicati-error.log
    CustomLog ${APACHE_LOG_DIR}/duplicati-access.log combined
</VirtualHost>
```