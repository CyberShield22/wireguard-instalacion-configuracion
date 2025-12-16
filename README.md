# Implantación de un sistema monitorización de red y análisis de logs en tiempo real

En este proyecto se ha diseñado e implementado un sistema completo de monitorización y análisis de logs con el objetivo de centralizar la observabilidad y el control de la infraestructura de red en un entorno basado en **Ubuntu Server**.

La implementación incluye la configuración de servicios fundamentales como **DHCP**, **DNS (BIND)**, **NGINX** como **proxy inverso**, y la activación de **enrutamiento y NAT** mediante **iptables**, garantizando conectividad, resolución de nombres y segmentación de red adecuadas para los distintos nodos del sistema.

Sobre esta infraestructura se ha desplegado la **pila ELK (Elasticsearch, Logstash y Kibana)** junto con **Filebeat** como agente de envío de logs, permitiendo la recolección, indexación y visualización en tiempo real de los registros generados por los diferentes equipos de la red (servidores y clientes).

El sistema se ha configurado de modo que el servidor central actúe como punto único de monitoreo, mientras que los nodos cliente (Ubuntu Profesor y Ubuntu Alumno) transmiten sus logs al stack ELK para su posterior análisis.

De esta forma, se logra una plataforma capaz de ofrecer **visibilidad completa sobre el tráfico, el rendimiento y los eventos del sistema**, además de **facilitar la detección temprana de incidencias y anomalías**.

La solución combina buenas prácticas de administración de sistemas Linux, gestión de red y herramientas de observabilidad modernas, constituyendo una base sólida para entornos DevOps o de seguridad en la nube.

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y/image.png)

# 1. Ubuntu server

## 1.1 Configuración Netplan

Se editará el archivo de configuración en YAML, llamado “00-installer-config.yaml”
que se encuentra en el directorio “/etc/netplan/”. Para poder editar este archivo se
ejecutará el siguiente comando:

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%201.png)

Se aplicará la siguiente configuración para configurar las dos tarjetas de red que
tiene el Ubuntu Server. El primer adaptador se ha configurado para que sea externo
y obtenga internet desde afuera, mientras que el segundo adaptador se ha
configurado para administrar la red local:

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%202.png)

## 1.2 DHCP

### 1.2.1 Instalación del servidor DHCP

Instalar un servidor DHCP es vital para gestionar y asignar dinámicamente
direcciones IP a los dispositivos que se conectan a la red. Esto implica la gestión de
la red y asegurar que los dispositivos puedan conectarse fácilmente a Internet, para
ello se ejecutará los siguientes comandos para proceder con la instalación:

```bash
sudo apt update
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%203.png)

```bash
sudo apt install isc-dhcp-server
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%204.png)

### 1.2.2 Configuración del servidor DHCP

Una vez instalado, ahora debemos de modificar el siguiente archivo de configuración
“/etc/dhcp/dhcpd.conf” para poder definir un rango de direcciones IP y otras opciones
como la puerta de enlace y los servidores DNS con el siguiente comando:

```bash
sudo nano /etc/dhcp/dhcpd.conf
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%205.png)

Donde vamos a implementar la siguiente configuración, para poder asegurarnos de
que las direcciones IP se asignan dinámicamente dentro del rango especificado, los
clientes usen la puerta de enlace “192.168.50.199”, que es la dirección del servidor,
los servidores DNS configurados son los de Google y un host específico con la
siguiente dirección MAC “08:00:27:19:11:b7”, que va a recibir la dirección IP
“192.168.50.200”, que en este caso este host será utilizado por el profesor del aula:

- **Definición de la Subred**:
    - Subred: “192.168.50.0”
    - Máscara de red: “255.255.255.0”
- **Rango de direcciones IP**:
    - Rango asignado: “192.168.50.200” a “192.168.50.250”
- **Puerta de enlace**:
    - IP de la puerta de enlace: “192.168.50.199”
- **Servidores DNS**:
    - DNS primario: “8.8.8.8” (Servidor DNS primario de Google)
    - DNS secundario: “8.8.4.4” (Servidor DNS secundario de Google)
- Nombre de Dominio:
    - Dominio: “davidpf.com”
- Tiempo de Arrendamietno:
    - Por defecto: “600 segundos” (10 minutos)
    - Máximo: “7200 segundos” (2 horas)
- Asignación Fija de IP para un Host Específico
    - Host: “ubuntu-profesor”
    - Dirección MAC: “08:00:27:19:11:b7”
    - IP fija: “192.168.50.200”

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%206.png)

Ahora ya configurado el archivo de configuración principal del servidor DHCP,
debemos de editar el archivo “/etc/default/isc-dhcp-server”, para poder especificar la
interfaz en la que el servidor DHCP debe escuchar:

```bash
sudo nano /etc/default/isc-dhcp-server
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%207.png)

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%208.png)

Una vez realizado lo anterior, se procede a reiniciar el servidor DHCP para que se
apliquen los cambios, con el siguiente comando:

```bash
sudo systemctl restart isc-dhcp-server
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%209.png)

Para poder verificar el funcionamiento del servidor DHCP, debemos de conectar un
cliente a la red y verificar que reciba una dirección IP del rango configurado, vamos a
realizar la comprobación en el Ubuntu Cliente que obtendría el profesor, ejecutamos
el siguiente comando para poder comprobarlo:

```bash
ip a
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2010.png)

## 1.3 Configuración del Enrutamiento y NAT

Para que los clientes puedan acceder a Internet, debemos de configurar el enrutamiento y habilitar NAT (Network Address Translation).

### 1.3.1 Habilitación del reenvío de IP

Para poder habilitar el reenvío de Ip, debemos de editar el archivo “/etc/sysctl.conf”,
con el siguiente comando:

```bash
sudo nano /etc/sysctl.conf
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2011.png)

Una vez, dentro del editor nano, debemos de descomentar la siguiente línea:

```bash
net.ipv4.ip_forward=1
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2012.png)

Para que se apliquen los cambios, debemos de ejecutar el siguiente comando:

```bash
sudo sysctl -p
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2013.png)

### 1.3.2 Configuración de iptables para NAT

Para poder configurar el iptables para NAT, debemos de ejecutar una serie de
comandos, para que añadan unas reglas, nuestra interfaz interna es “enp0s8” y la
interfaz externa que es la que está conectada a Internet es “enp0s3”:

```bash
sudo iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE
sudo iptables -A FORWARD -i enp0s8 -o enp0s3 -j ACCEPT
sudo iptables -A FORWARD -i enp0s3 -o enp0s8 -m state –state
RELATED,ESTABLISHED -j ACCEPT
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2014.png)

### 1.3.3 Instalación del paquete “iptables-persistent”

Para que las reglas de iptables persistan después de reiniciar el servidor, debemos
de instalar el paquete llamado “iptables-persistent”, ejecutando los siguientes
comandos:

```bash
sudo apt install iptables-persistent
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2015.png)

Una vez que lo hayamos instalado, nos aparecerá una ventana, donde nos preguntará que si queremos guardar las reglas de IPv4 actuales y nosotros le indicaremos que sí.

Luego nos aparecerá otra vez la misma pantalla, pero en vez de IPv4, nos preguntarán por las reglas IPv6 y vamos a marcar que si, como lo hemos hecho anteriormente:

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2016.png)

### 1.3.4 Guardar las reglas actuales de iptables

Para poder asegurarnos de que las reglas actuales de iptables se guarden y se restauren después de un reinicio, se ejecutará el siguiente comando:

```bash
sudo netfilter-persistent save
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2017.png)

Este comando ha guardado las reglas actuales en archivos de configuración
localizados en “/etc/iptables/rules.v4” para IPv4 y “/etc/iptables/rules.v6” para IPv6.

### 1.3.5 Recargar las reglas guardadas de iptables

Para poder aplicar las reglas guardadas, por ejemplo, después de modificarlas
manualmente en los archivos de configuración, se ejecutará el siguiente comando:

```bash
sudo netfilter-persistent reload
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2018.png)

## 1.4 Java con APT

Instalar Java de manera fácil es posible aprovechando la versión incluida en el
paquete estándar de Ubuntu. De forma predeterminada, Ubuntu 20.04 viene
equipado con Open JDK 11, una versión de JRE y JDK de fuente abierta.

### 1.4.1 Instalación de JRE y del JDK predeterminado

Para poder instalar esta versión, primero se actualizará el índice de paquetes,
ejecutando el siguiente comando:

```bash
sudo apt update
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2019.png)

Para poder comprobar si Java ya está instalado, se ejecutará el siguiente comando:

```bash
java -version
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2020.png)

En caso de que Java no esté instalado, obtendremos el siguiente resultado:

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2021.png)

Para instalar Java Runtime Environment (JRE) predeterminado, se ejecutará el
siguiente comando:

```bash
sudo apt install default-jre
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2022.png)

Verificamos la instalación con el siguiente comando:

```bash
java -version
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2023.png)

Como podemos observar, en la imagen anterior, se ha realizado con éxito la
instalación. Es posible que se necesite el kit de desarrollo de Java (JDK) además de
JRE para compilar y ejecutar algunos programas específicos basados en Java, para
instalar JDK, se ejecutará el siguiente comando:

```bash
sudo apt install default-jdk
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2024.png)

## 1.5 DNS

Un aspecto crucial en la gestión de la configuración y la infraestructura de los
servidores es la implementación constante de un método fácil para poder localizar
interfaces de red y direcciones IP por nombre, a través de la configuración de un
sistema de nombres de dominio (DNS) apropiado. Utilizar nombres de dominio
completos (FQDN) en lugar de direcciones IP para definir las direcciones de red
puede simplificar la configuración de servicios y aplicaciones, y mejora la
mantenibilidad de los archivos de configuración.

### 1.5.1 Instalación de BIND

Utilizaremos el software de servidor de nombres BIND (BIND9), para poder
instalarlo, debemos de de ejecutar el siguiente comando:

```bash
sudo apt-get install bind9 bind9utils bind9-doc
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2025.png)

### 1.5.2 Configuración de BIND

La configuración de BIND se compone de múltiples archivos que se incorporan
desde el archivo principal de configuración “named.conf”. Estos archivos llevan el
prefijo named, ya que corresponde al nombre del proceso que ejecuta BIND (una
abreviatura de “domain name daemon”). Primero vamos a configurar el archivo de
opciones.
Abrimos el archivo “named.conf.options” con el editor nano, ejecutando el siguiente
comando:

```bash
sudo nano /etc/bind/named.conf.options
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2026.png)

Sobre el bloque “options” que hay, debemos de crear un nuevo bloque ACL, que es
una lista de control de acceso llamado “trusted”, donde definiremos una lista de
clientes desde los que permitiremos consultas de DNS recurrentes y luego en el
bloque “options”, debemos de añadir las siguientes líneas que están marcadas:

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2027.png)

Una vez que hayamos agregado las líneas, guardamos y cerramos editor nano,
después tenemos que configurar el archivo local para poder especificar nuestras
zonas de DNS.

Abrimos el archivo “named.conf.options” con el siguiente comando:

```bash
sudo nano /etc/bind/named.conf.local
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2028.png)

El archivo debe de estar vacío, aquí es donde vamos a especificar nuestras zonas de reenvío e inversas. Las zonas DNS establecen un rango específico para gestionar y definir registros DNS. Como nuestro dominio es “[davidpf.com](http://davidpf.com/)”, vamos a utilizarlo como nuestra zona de reenvío. Dado que las direcciones IP privadas de nuestros servidores se encuentran en el espacio IP “192.168.50.0/24”, configuraremos una zona inversa para poder realizar búsquedas inversas en ese rango, por lo tanto, lo hemos configurado de la siguiente manera:

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2029.png)

Una vez realizado lo anterior, ahora debemos de crear el archivo de la zona de
reenvío, porque el archivo de la zona de reenvío es donde establecemos los
registros DNS para redirigir las consultas DNS. Esto significa que cuando el DNS
recibe una consulta de nombre, como “[ubuntualu.davidpf.com](http://ubuntualu.davidpf.com/)”, buscará en el
archivo de la zona de reenvío para poder determinar la dirección IP privada
correspondiente a Windows Cliente.
Pero antes debemos de crear el directorio en el que se alojarán nuestros archivos de
zona, lo creamos con el siguiente comando:

```bash
sudo mkdir /etc/bind/zones
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2030.png)

Vamos a basarnos en nuestro archivo de la zona de reenvío en el archivo de zona
“db.local” de muestra, lo copiamos:

```bash
sudo cp /etc/bind/db.local /etc/bind/zones/db.davidpf.com
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2031.png)

Una vez copiado, lo editamos con el editor nano:

```bash
sudo nano /etc/bind/zones/db.davidpdf.com
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2032.png)

Y lo configuramos de la siguiente manera:

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2033.png)

Una vez realizado la configuración, ahora debemos de crear los archivos de la zona,
en el que estableceremos los registros DNS PTR para poder realizar búsquedas
inversas de DNS, esto quiere decir que cuando el DNS recibe una consulta por una
dirección IP, como “192.168.50.205”, por ejemplo, consultará los archivos de la zona
inversa para determinar el FQDN correspondiente, que en este caso sería
“[windowsalu.davidpf.com](http://windowsalu.davidpf.com/)”.
Realizamos lo mismo que anteriormente, pero con el fichero de zonas inversa:

```bash
sudo cp /etc/bind/db.127 /etc/bind/zones/db.50.168.192
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2034.png)

Editamos el archivo con el editor nano:

```bash
sudo nano /etc/bind/zones/db.50.168.192
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2035.png)

Y lo configuramos de la siguiente manera:

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2036.png)

Guardamos y cerramos el editor nano, para poder comprobar que todo está bien
escrito, podemos ejecutar el siguiente comando:

```bash
named-checkconf
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2037.png)

Si no nos muestra nada, es que todo bien escrito.

Y en caso, de que queramos comprobar la configuración de la zona de reenvío
“[davidpf.com](http://davidpf.com/)”, ejecutamos el siguiente comando:

```bash
sudo named-checkzone davidpf.com /etc/bind/zones/db.davidpf.com
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2038.png)

Como podemos comprobar, está todo OK y para comprobar la configuración de la
zona inversa, ejecutamos el siguiente comando:

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2039.png)

Una vez terminado, procedemos a reiniciar BIND con el siguiente comando:

```bash
sudo systemctl restart bind9
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2040.png)

### 1.5.3 Configuración clientes DNS

Debemos de dirigirnos al Ubuntu del Profesor, primero comprobamos si tiene una
dirección IP de nuestro servidor DHCP, con el siguiente comando:

```bash
ip a
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2041.png)

Una vez comprobado, debemos de dirigirnos al archivo de configuración de la tarjeta
de red y debemos de añadir la dirección de nuestro servidor DNS:

```bash
sudo nano /etc/netplan/01-network-manager-all.yaml
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2042.png)

Aplicamos los cambios del netplan, con el siguiente comando:

```bash
sudo netplan apply
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2043.png)

Después, para asegurarnos de que el cliente pueda hacer ping al servidor utilizando
el nombre del dominio, vamos a dirigirnos al siguiente fichero “/etc/hosts”, con el
siguiente comando:

```bash
sudo nano /etc/hosts
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2044.png)

Y vamos a agregar las dos siguientes líneas:

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2045.png)

## 1.6 NGINX

NGINX se destaca como uno de los servidores web líderes a nivel global, hospedando sitios de alta demanda y tráfico intenso en la red. Su diseño eficiente lo hace ideal tanto para funcionar como servidor web como para actuar de proxy inverso.

### 1.6.1 Instalación de NGINX

Gracias a que NGINX, está disponible en los repositorios predeterminados de
Ubuntu, se puede instalarse desde estos repositorios usando el sistema de paquetes
“apt”. Se ejecutará el siguiente comando, para realizar la instalación:

```bash
sudo apt install nginx
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2046.png)

### 1.6.2 Aplicación de ajustes al firewall

Es esencial configurar el software del firewall antes de iniciar Nginx, para asegurar el acceso al servicio. Tras su instalación, NGINX se añade automáticamente a ufw
como un servicio, facilitando así la autorización de su acceso.

Para poder enumerar las configuraciones de la aplicación con las que “ufw” sabe trabajar, se ejecutará el siguiente comando:

```bash
sudo afw app list
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2047.png)

Como se muestra en el resultado, hay tres perfiles disponibles para NGINX:

- **NGINX Full**: este perfil abre el puerto 80 (tráfico web normal, no cifrado) y el puerto 443 (tráfico TLS / SSL cifrado)
- **NGINX HTTP**: este perfil abre solo el puerto 80 (tráfico web normal, no
cifrado)
- **NGINX HTTPS**: este perfil abre solo el puerto 443 (tráfico TLS/SSL cifrado)

Es aconsejable activar el perfil de seguridad más estricto, el cual aún así facilitará el
paso del tráfico previamente establecido. Actualmente, es suficiente con autorizar el
tráfico a través del puerto 80. Se ejecutará el siguiente comando para poder
habilitarlo:

```bash
sudo afw allow 'Nginx HTTP'
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2048.png)

Para comprobar el cambio, ejecutamos el siguiente comando:

```bash
sudo ufw status
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2049.png)

### 1.6.3 Comprobación de su servidor web

Para poder realizar una verificación de que el servicio esté en ejecución, vamos a ejecutar el siguiente comando:

```bash
systemctl status nginx
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2050.png)

Como podemos comprobar, el servicio se inició correctamente, sin duda, la mejor forma para comprobarlo es solicitando una página de Nginx, para ello, debemos de dirigirnos al Ubuntu Cliente del profesor y dirigirnos a la dirección IP de nuestro servidor:

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2051.png)

### 1.6.4 Configuración para convertirlo en proxy inverso

Primero debemos de dirigirnos al siguiente fichero “/etc/nginx/sites-available/default”
y debemos de añadir el siguiente bloque de código al archivo:

```bash
sudo nano /etc/nginx/sites-available/default
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2052.png)

Debemos de añadir el siguiente bloque de código al archivo:

```bash
server {
	listen 80;
	server_name your_domain;
	auth_basic "Restricted Access";
	auth_basic_user_file /etc/nginx/htpasswd.users;
	location / {
		proxy_pass http://localhost:5601;
		proxy_http_version 1.1;
		proxy_set_header Upgrade $http_upgrade;
		proxy_set_header Connection 'upgrade';
		proxy_set_header Host $host;
		proxy_cache_bypass $http_upgrade;
	}
}
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2053.png)

Guardamos y cerramos el editor nano. Para poder evitar un problema de memoria
de depósito de hash que pueda surgir al agregar nombres de servidor, es necesario
aplicar ajustes a un valor en el archivo “etc/nginx/nginx.conf”, lo abrimos con el editor
nano, ejecutando el siguiente comando:

```bash
sudo nano /etc/nginx/nginx.conf
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2054.png)

Debemos de encontrar la siguiente directiva “server_names_hash_bucket_size” y
borrar el símbolo para poder descomentar la línea, una vez realizado esto,
guardamos el archivo y cerramos el editor nano:

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2055.png)

Para poder comprobar de que no hayan errores sintaxis en ninguno de sus archivos
de NGINX, podemos ejecutar el siguiente comando:

```bash
sudo nginx -t
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2056.png)

Como hemos podido comprobar, no ha habido ningún error de sintaxis, procedemos
a reiniciar NGINX para que se habiliten los cambios:

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2057.png)

## 1.7 ELK Stack

### 1.7.1 Instalación de ElasticSearch

Cada paquete de ElasticSearch viene con una firma de seguridad propia para
salvaguardar el sistema de cualquier intento de falsificación de paquetes. Nuestro
gestor de paquetes confiará en aquellos paquetes que estén autenticados con esta
firma. En la siguiente parte, procederemos a importar la clave pública GPC de
ElasticSearch y añadiremos el repositorio de paquetes de Elastic para la instalación
de ElasticSearch.

Para importar la clave GPC pública de ElasticSearch a APT, debemos de utilizar la
herramienta llamada “curl”, que es una herramienta de línea de comandos para
transferir datos con URL, lo hacemos con el siguiente comando:

```bash
curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo aptkey
add -
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2058.png)

Después, agregaremos la lista de fuentes de Elastic al directorio “sources.list.d”,
donde APT buscará nuevas fuentes:

```bash
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" |
sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2059.png)

A continuación, se actualizará las listas de paquetes para que APT lea la nueva
fuente de Elastic:

```bash
sudo apt update
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/48a9882a-b3ff-43d0-8dd2-d82c860b171d.png)

Procedemos a instalar ElasticSearch con el siguiente comando:

```bash
sudo apt install elasticsearch
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2060.png)

Ahora ya que tenemos instalado ElasticSearch instalado, vamos a utilizar el editor
nano, para poder modificar el archivo de configuración principal de ElasticSearch
“elasticsearch.yml”, este archivo puede llegar a ofrecer opciones de configuración
para su clúster, nodo, rutas, memoria, red, detección y puerta de enlace.

La mayor parte de las configuraciones ya vienen establecidas por defecto en el
archivo, pero solamente nos centraremos exclusivamente en ajustar la
configuración del anfitrión de la red.

ElasticSaerch escucha el tráfico de todos los lugares en el puerto “9200”. Es
conveniente restringir el acceso externo a su instancia de ElasticSearch para evitar
que terceros lean sus datos o cierren nuestro clúster de ElasticSearch a través de su
[API REST].

Para poder restringir el acceso y aumentar la seguridad, vamos a buscar la línea
que especifica “network.host”, eliminamos los comentarios y reemplazamos su valor
por “localhost”, se debe de quedar de la siguiente manera:

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2061.png)

Hemos especificado “localhost” para que Elasticsearch escuche en todas las
interfaces y las IP vinculadas. Si desea que escuche únicamente en una interfaz
específica, puede especificar su IP en lugar de localhost.

Guardamos y cerramos el editor nano. Ahora ya si que podemos iniciar
ElasticSearch, con el siguiente comando:

```bash
sudo systemctl start elasticsearch
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2062.png)

Luego, ejecutamos el siguiente comando para permitir que ElasticSearch se cargue
cada vez que nuestro servidor se inicie:

```bash
sudo systemctl enable elasticsearch
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2063.png)

Para comprobar si ElasticSearch, se está ejecutando enviando una solicitud HTTP,
debemos de ejecutar el siguiente comando, en el que nos mostrará la siguiente
respuesta, en el que nos indica que está configurado y activo:

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2064.png)

### 1.7.2 Instalación de Kibana

Ahora ya que tenemos en funcionamiento ElasticSearch, ahora debemos de instalar
Kibana, podemos hacerlo con el siguiente comando:

```bash
sudo apt install kibana
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2065.png)

Habilitamos e iniciamos el servicio Kibana ejecutando los siguientes comandos:

```bash
sudo systemctl enable kibana
sudo systemctl start kibana
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2066.png)

Ya que Kibana se encuentra configurado para recibir conexiones únicamente desde
localhost, necesitamos establecer un proxy inverso para habilitar el acceso desde
fuera. Para lograr esto, haremos uso de NGINX, el cual ya tenemos instalado en el
servidor.

Primero, vamos a utilizar el comando “openssl”, para poder crear un usuario
administrativo de Kibana que se usará para poder acceder a la interfaz web de
Kibana. Con el siguiente comando se crearán el usuario y la contraseña
administrativa de Kibana y se almacenarán en el archivo “htpasswd.users”:

```bash
echo "davidpf:`openssl passwd -apr1`" | sudo tee -a
/etc/nginx/htpasswd.users
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2067.png)

Como anteriormente ya hemos hecho las configuraciones necesarias para que
NGINX funcione como proxy inverso, solamente nos queda dirigirnos al Ubuntu del
profesor, abrir el Firefox y buscar lo siguiente:

```bash
http://davidpf.com/status
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2068.png)

Y como podemos comprobar en la anterior imagen, hemos podido acceder con éxito
y también en la imagen se muestra información sobre el uso de los recursos del
servidor y se enumeran los complementos instalados.

### 1.7.3 Instalación y configuración de LogStash

A pesar de que Beats tiene la capacidad de transmitir datos directamente a
ElasticSearch, sugerimos utilizar LogStash para el procesamiento de datos. Esto
ofrece una mayor flexibilidad al permitirte recolectar datos de diversas fuentes,
convertirlos a un formato unificado y luego exportarlos a otra base de datos.

Instalamos Logstash ejecutando el siguiente comando:

```bash
sudo apt install logstash
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2069.png)

Una vez que hayamos instalado Logstash, podemos continuar con su configuración.
Los archivos de configuración de Logstash se encuentran en el directorio
“/etc/logstash/conf.d”. Para más detalles sobre la sintaxis de configuración, podemos
referirnos a la guía de configuración que nos proporcionan Elastic. Al configurar el
archivo, puede ser útil considerar que Logstash es un proceso que recibe datos en
un extremo, los procesa de alguna manera y los envía a ElasticSearch. Un proceso
de Logstash consta de dos elementos esenciales, “input” y “output”, y un elemento
opcional “filter”. Los plugins de entrada recogen datos de una fuente, los plugins de
filtro procesan los datos, y los plugins de salida escriben los datos en un destino.

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2070.png)

Primero vamos a crear un archivo de configuración llamado “02-beats-input.conf” en
el que vamos a establecer la entrada de “Filebeat”:

```bash
sudo nano /etc/logstash/conf.d/02-beats-input.conf
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2071.png)

Donde vamos a introducir la siguiente configuración de “input”, donde se
especificará una entrada de “beats” que escuchará en el puerto TCP “5044”:

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2072.png)

Guardamos y cerramos el editor nano, creamos otro archivo de configuración
llamado “30-elasticsearch-output.conf”:

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2073.png)

Vamos a ingresar la siguiente configuración de salida. En esencia, con esta salida,
se configura Logstash para almacenar los datos de Beats en Elasticsearch, que se
encuentra en localhost:9200, en un índice con el nombre del Beat que se está
utilizando. El Beat que se utiliza en este tutorial es Filebeat:

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2074.png)

Una vez que hayamos terminado escribir la configuración, guardamos y cerramos el
editor nano y comprobamos si la configuración de logstash funciona:

```bash
sudo -u logstash /usr/share/logstash/bin/logstash --path.settings
/etc/logstash -t
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2075.png)

Como podemos comprobar en la imagen, nos han salido varios errores de
OpenJDK, pero no deberían causar ningún problema, se pueden ignorar, lo
importante es la última que debe ser “Validation Result: OK. Exiting Logstash”

Iniciamos y habilitamos Logstash para poder implementar los cambios de
configuración:

```bash
sudo systemctl start logstash
sudo systemctl enable logstash
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2076.png)

### 1.7.4 Instalación y configuración de Filebeat

La suite de Elastic incorpora una serie de agentes de transporte eficientes conocidos
como Beats, que se encargan de recolectar información de distintas fuentes para su
posterior envío a Logstash o Elasticsearch. Los siguientes son los Beats disponibles
actualmente en Elastic:

- **Filebeat**: Se especializa en la recolección y transmisión de archivos de registro.
- **Metricbeat**: Se encarga de recoger métricas de sistemas y servicios.
- **Packetbeat**: Se dedica a la recolección y análisis de tráfico de red.
- **Winlogbeat**: Se ocupa de recopilar los registros de eventor del sistema operativo Windows.
- **Auditbeat**: Recoge datos de auditoría del sistema Linux y realiza seguimiento de la integridad de los archivos.
- **Heartbeat**: Monitorea la disponibilidad de servicios mediante comprobaciones activas.

Nosotros, vamos a instalar Filebeat para que reenvíe registros locales a nuestra pila
de Elastic, lo instalamos con el siguiente comando:

```bash
sudo apt install filebeat
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2077.png)

Después de haber realizado la instalación, debemos configurar Fliebeat para que se
conecte a Logstash, para ello debemos de modificar el archivo de configuración de
ejemplo que viene con Filebeat, lo abrimos con el editor nano:

```bash
sudo nano /etc/filebeat/filebeat.yml
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2078.png)

Filebeat es compatible con una variedad de destinos para los eventos que recopila.
Comúnmente, estos eventos se envían ya sea directamente a Elasticsearch o a
Logstash para un procesamiento más detallado. Ahora vamos a optar por utilizar
Logstash para añadir una capa adicional de procesamiento a la información que
Filebeat ha reunido. Dado que Filebeat no necesitará realizar envíos directos a
Elasticsearch, procederemos a deshabilitar esta función. Para ello, localiza la
sección “output.elasticsearch” en el archivo de configuración y comenta las líneas
pertinentes añadiendo un # al inicio de cada una:

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2079.png)

Después, debemos configurar “output.logstash”, para ello debemos de eliminar el
comentario de las líneas “output.logstash:” y “hosts: [“localhost:5044”]” quitando “#”,
con esto conseguiremos que se configure Filebeat para poder establecer conexión
con logstash en su servidor ElasticStack en el puerto “5044”:

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2080.png)

Guardamos y cerramos el editor nano. La versatilidad de Filebeat se potencia
mediante el uso de sus [módulos específicos](https://www.elastic.co/guide/en/beats/filebeat/7.6/filebeat-modules.html). En la presente guía, implementaremos
el módulo de sistema, diseñado para recolectar y procesar los logs generados por el
sistema de registro integrado en las distribuciones de Linux más utilizadas.

Para poder habilitarlo, debemos de ejecutar el siguiente comando:

```bash
sudo filebeat modules enable system
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2081.png)

También podremos ver la lista de módulos que tenemos habilitados y desactivados
ejecutando el siguiente comando (me he conectado por ssh al servidor, desde el
ubuntu del profesor para poder ver la lista completa):

```bash
sudo filebeat modules list
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2082.png)

De manera predeterminada, Filebeat viene preconfigurado para seguir las rutas
estándar de los archivos de registro syslog y de autorización. Para los propósitos de
esta guía, no será necesario realizar ajustes en estas configuraciones. Los detalles
específicos del módulo se encuentran disponibles para su consulta en el archivo
/etc/filebeat/modules.d/system.yml.

A continuación, es necesario establecer los procesadores de ingesta de Filebeat,
responsables de examinar los registros antes de su envío a Elasticsearch mediante
Logstash. Para implementar el procesador de ingesta asociado al módulo de
sistema, ejecutamos el comando que se indica a continuación:

```bash
sudo filebeat setup --pipelines --modules system
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2083.png)

A continuación, procederemos a cargar la plantilla de índice en Elasticsearch. Un
índice en Elasticsearch representa una colección de documentos con atributos
semejantes. Cada índice se distingue por un nombre único, el cual se emplea para
referenciar al índice durante la ejecución de operaciones diversas. Al instaurar un
índice nuevo, la plantilla de índice correspondiente se asignará de manera
automática, para poder hacerlo, vamos a ejecutar el siguiente comando:

```bash
sudo filebeat setup --index-management -E
output.logstash.enabled=false -E
'output.elasticsearch.hosts=["localhost:9200"]'
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2084.png)

Filebeat ofrece paneles demostrativos en Kibana que facilitan la visualización de los
datos de Filebeat dentro de la misma plataforma. Para utilizar estos paneles, es
necesario establecer el patrón de índice y proceder con la carga de los paneles en
Kibana.

Durante la carga de los paneles, Filebeat realiza una conexión con Elasticsearch
para confirmar los detalles de la versión. Si desea cargar los paneles mientras
Logstash está activo, necesitará deshabilitar la salida de Logstash y activar la salida
hacia Elasticsearch:

```bash
sudo filebeat setup -E output.logstash.enabled=false -E
output.elasticsearch.hosts=['localhost:9200'] -E
setup.kibana.host=localhost:5601
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2085.png)

Una vez, realizado todo lo anterior, ahora ya si que podremos iniciar y habilitar
Filebeat, con los siguiente comandos:

```bash
sudo systemctl start filebeat
sudo systemctl enable filebeat
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2086.png)

Si se ha configurado de la manera correcta, Filebeat comenzará el envío de sus
registros syslog y de autorización a Logstash, que a su vez cargará esos datos en
ElasticSearch.

Para pode verificar que ElasticSearch realmente esté recibiendo estos datos,
debemos de consultar el índice de Filebeat con el siguiente comando:

```bash
curl -XGET 'http://localhost:9200/filebeat-*/_search?pretty'
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2087.png)

Como podemos comprobar ya está todo OK, podemos comprobar que ElasticSearch
está cargando registros bajo el índice que hemos buscado y ahora debemos de
diriginos a ubuntu del profesor, para poder visualizar.

## 2. Ubuntu Profesor

## 2.1 Configuración Netplan

Se editará el archivo de configuración en YAML, llamado “01-network-managerall.
yaml” que se encuentra en el directorio “/etc/netplan/”. Para poder editar este
archivo se ejecutará el siguiente comando:

```bash
sudo nano /etc/netplan/ 01-network-manager-all.yaml
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2088.png)

Se aplicará la siguiente configuración para configurar la tarjeta de red que tiene el
Ubuntu Desktop, que este lo utilizará el profesor. El primer adaptador se ha
configurado para obtenga una dirección IP reservada del servidor DHCP y también
utilice el servidor DNS nuestro:

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2089.png)

## 2.2 Configuración /etc/hosts

El archivo “/etc/hosts” es un archivo de configuración que se utiliza para mapear
nombres de host a direcciones IP, su objetivo principal es permitir que el sistema
resuelva nombres de dominio a direcciones IP sin tener que consultar un servidor
DNS, lo editamos con el editor nano, ejecutando el siguiente comando:

```bash
sudo nano /etc/hosts
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2090.png)

Donde debemos de agregar las dos siguientes líneas:

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2091.png)

Una vez, realizado esto, actualizamos el adaptador de red con el siguiente comando,
para que se apliquen los cambios:

```bash
sudo netplan apply
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2092.png)

Y ya lo tendríamos todo configurado:

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2093.png)

## 2.3 Acceso a Elastic

Para poder acceder a ElasticSearch, debemos de escribir “[http://davidpf.com](http://davidpf.com/)” en el
navegador:

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2094.png)

## 2.4 Acceso a Kibana

Para poder acceder a ElasticSearch, debemos de escribir “[http://davidpf.com/status”](http://davidpf.com/status%E2%80%9D)
en el navegador:

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs/image%2095.png)

# 3. Ubuntu Alumno

## 3.1 Configuración Netplan

Se editará el archivo de configuración en YAML, llamado “01-network-managerall.
yaml” que se encuentra en el directorio “/etc/netplan/”. Para poder editar este
archivo se ejecutará el siguiente comando:

```bash
sudo nano /etc/netplan/ 01-network-manager-all.yaml
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs%202/image%2096.png)

Se aplicará la siguiente configuración para configurar la tarjeta de red que tiene el
Ubuntu Desktop, que este lo utilizará el profesor. El primer adaptador se ha
configurado para obtenga una dirección IP reservada del servidor DHCP y también
utilice el servidor DNS nuestro:

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs%202/image%2097.png)

## 3.2 Configuración /etc/hosts

El archivo “/etc/hosts” es un archivo de configuración que se utiliza para mapear
nombres de host a direcciones IP, su objetivo principal es permitir que el sistema
resuelva nombres de dominio a direcciones IP sin tener que consultar un servidor
DNS, lo editamos con el editor nano, ejecutando el siguiente comando:

```bash
sudo nano /etc/hosts
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs%202/image%2098.png)

Donde debemos de agregar las dos siguientes líneas:

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs%202/image%2099.png)

Una vez, realizado esto, actualizamos el adaptador de red con el siguiente comando,
para que se apliquen los cambios:

```bash
sudo netplan apply
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs%202/image%20100.png)

Y ya lo tendríamos todo configurado:

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs%202/image%20101.png)

## 3.3 Instalación de FileBeat

Para poder descargar FileBeat, debemos de dirigirnos a la página oficial Elastic, elegimos el
sistema operativo, en este caso elegimos Linux y descargamos:

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs%202/image%20102.png)

Después abrimos la terminal, nos dirigimos a Descargas:

```bash
cd Descargas/
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs%202/image%20103.png)

Una vez que estemos dentro de descargas, vamos a descomprimir el archivo que
hemos descargado anteriormente:

```bash
tar -xzf filebeat-8.14.0-linux-x86_64.tar.gz
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs%202/image%20104.png)

Accedemos a la directorio que acabamos de descomprimir de Filebeat y vamos a abrir el archivo “filebeat.yml” y vamos a repetir los mismo pasos que hemos hecho anteriormente en el servidor:

Se debe comentar las siguientes líneas:

```bash
#output.elasticsearch:
	# Array of hosts to connect to.
	#hosts: ["localhost:9200"]
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs%202/image%20105.png)

Se debe descomentar las siguientes líneas:

```bash
output.logstash:
	# The Logstash hosts
	hosts: ["localhost:5044"]
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs%202/image%20106.png)

Una vez, realizado los cambios, guardamos y cerramos el editor nano y lo
habilitamos con el siguiente comando:

```bash
./filebeat
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs%202/image%20107.png)

Para poder comprobar que todo está correcto, vamos a ejecutar el siguiente
comando:

```bash
sudo systemctl status
```

![image.png](Implantaci%C3%B3n%20de%20un%20sistema%20monitorizaci%C3%B3n%20de%20red%20y%20análisis%20de%20logs%202/image%20108.png)
