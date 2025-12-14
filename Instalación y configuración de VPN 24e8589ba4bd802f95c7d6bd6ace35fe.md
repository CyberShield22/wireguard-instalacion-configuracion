# Instalación y configuración de VPN

# 1. Instalación Wireguard

Antes de proceder con la instalación de Wireguard, debemos de ejecutar el siguiente comando para poder proceder con la actualización de los repositorios:

```
apt update
```

![Pasted image 20250819194846.png](Instalaci%C3%B3n%20y%20configuraci%C3%B3n%20de%20VPN/Pasted_image_20250819194846.png)

Una vez, que tengamos actualizados los repositorios, podemos proceder con la instalación de Wireguard ejecutando el siguiente comando:

```
apt install wireguard
```

![Pasted image 20250819194956.png](Instalaci%C3%B3n%20y%20configuraci%C3%B3n%20de%20VPN/Pasted_image_20250819194956.png)

# 2. Configuración del Servidor

Para poder proceder con la configuración de Wireguard, debemos de dirigirnos a la siguiente carpeta `/etc/wireguard` ejecutando el siguiente comando:

```
cd /etc/wireguard
```

![Pasted image 20250819195201.png](Instalaci%C3%B3n%20y%20configuraci%C3%B3n%20de%20VPN/Pasted_image_20250819195201.png)

Vamos a crearnos una carpeta llamada `claves_servidor`, donde está carpeta contendrá las claves pública y privada con el siguiente comando:

```
mkdir /etc/wireguard/claves_servidor
```

![Pasted image 20250819195703.png](Instalaci%C3%B3n%20y%20configuraci%C3%B3n%20de%20VPN/Pasted_image_20250819195703.png)

Accedemos dentro de la carpeta que acabamos de crear ejecutando el siguiente comando:

```
cd /etc/wireguard/claves_servidor
```

![Pasted image 20250819195802.png](Instalaci%C3%B3n%20y%20configuraci%C3%B3n%20de%20VPN/Pasted_image_20250819195802.png)

Una vez, que estemos dentro, ahora debemos de generar un par de claves (privada y pública) para el servidor para ello podemos hacerlo ejecutando el siguiente comando:

```
wg genkey | tee servidor_privada.key | wg pubkey > servidor_publica.key
```

![Pasted image 20250819195853.png](Instalaci%C3%B3n%20y%20configuraci%C3%B3n%20de%20VPN/Pasted_image_20250819195853.png)

Por temas de seguridad, vamos a modificar los permisos de esta carpeta para que solamente nosotros podemos editar los los archivos, ejecutamos el siguiente comando:

```
chmod 600 -R ../claves_servidor/
```

![Pasted image 20250821125326.png](Instalaci%C3%B3n%20y%20configuraci%C3%B3n%20de%20VPN/Pasted_image_20250821125326.png)

Ahora vamos a crear un archivo de configuración donde va a contener la información para nuestra VPN:

```
touch wg0.conf
```

![Pasted image 20250821125540.png](Instalaci%C3%B3n%20y%20configuraci%C3%B3n%20de%20VPN/Pasted_image_20250821125540.png)

Ahora para poder añadir nuestra clave privada de nuestro servidor en este archivo que acabamos de crear, debemos de ejecutar el siguiente comando:

```
cat claves_servidor/servidor_privada.key >> wg0.conf
```

![Pasted image 20250821125816.png](Instalaci%C3%B3n%20y%20configuraci%C3%B3n%20de%20VPN/Pasted_image_20250821125816.png)

Una vez realizado todo lo anterior, ahora editaremos el archivo `wg0.conf` con el editor nano, para ello vamos a ejecutar el siguiente comando:

```
nano wg0.conf
```

![Pasted image 20250821125946.png](Instalaci%C3%B3n%20y%20configuraci%C3%B3n%20de%20VPN/Pasted_image_20250821125946.png)

Dentro del archivo ya observamos la clave privada del servidor que hemos añadido anteriormente, donde ahora debemos de rellenarlo con las siguientes líneas para que se quede de la siguiente forma:

```
[Interface]
Address = 192.168.2.1
PrivateKey = <Clave Privada del servidor>
ListenPort = 51820
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o wlp1s0 -j MASQUERADE;
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o wlp1s0 -j MASQUERADE
```

![Pasted image 20250821141122.png](Instalaci%C3%B3n%20y%20configuraci%C3%B3n%20de%20VPN/Pasted_image_20250821141122.png)

Guardamos y cerramos el editor nano, después debemos de habilitar el servicio Wireguard ejecutando el siguiente comando:

```
systemctl enable wg-quick@wg0
```

![Pasted image 20250821141400.png](Instalaci%C3%B3n%20y%20configuraci%C3%B3n%20de%20VPN/Pasted_image_20250821141400.png)

Y lo habilitamos con el siguiente comando:

```
systemctl start wg-quick@wg0
```

![Pasted image 20250821141636.png](Instalaci%C3%B3n%20y%20configuraci%C3%B3n%20de%20VPN/Pasted_image_20250821141636.png)

Para poder comprobar que el servicio está corriendo correctamente, podemos hacerlo ejecutando el siguiente comando:

```
systemctl status wg-quick@wg0
```

![Pasted image 20250821141803.png](Instalaci%C3%B3n%20y%20configuraci%C3%B3n%20de%20VPN/Pasted_image_20250821141803.png)

Y para poder comprobar si se ha creado una nueva interfaz para nuestra VPN, podemos saberlo ejecutando el siguiente comando:

```
ifconfig
```

![Pasted image 20250821172953.png](Instalaci%C3%B3n%20y%20configuraci%C3%B3n%20de%20VPN/Pasted_image_20250821172953.png)

Para que los clientes que estén conectados a la VPN puedan acceder   a Internet, debemos de habilitar el reenvío IP con los siguientes comandos:

```
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
sysctl -p
```

![Pasted image 20250901204222.png](Instalaci%C3%B3n%20y%20configuraci%C3%B3n%20de%20VPN/Pasted_image_20250901204222.png)

# 3. Configuración Cliente

## 3.1 Android

Para tenerlo todo más organizado vamos a crear una carpeta llamada `clientes` con el siguiente comando:

```
mkdir clientes
```

![Pasted image 20250821173156.png](Instalaci%C3%B3n%20y%20configuraci%C3%B3n%20de%20VPN/Pasted_image_20250821173156.png)

![Pasted image 20250821173211.png](Instalaci%C3%B3n%20y%20configuraci%C3%B3n%20de%20VPN/Pasted_image_20250821173211.png)

Accedemos a la carpeta `clientes` con el siguiente comando:

```
cd clientes
```

![Pasted image 20250821173258.png](Instalaci%C3%B3n%20y%20configuraci%C3%B3n%20de%20VPN/Pasted_image_20250821173258.png)

Una vez, que estemos dentro, vamos a crearnos otra carpeta llamada `android`:

```
mkdir android
```

![Pasted image 20250821173412.png](Instalaci%C3%B3n%20y%20configuraci%C3%B3n%20de%20VPN/Pasted_image_20250821173412.png)

![Pasted image 20250821173425.png](Instalaci%C3%B3n%20y%20configuraci%C3%B3n%20de%20VPN/Pasted_image_20250821173425.png)

Accedemos a la carpeta `android` con el siguiente comando:

```
cd android
```

![Pasted image 20250901184652.png](Instalaci%C3%B3n%20y%20configuraci%C3%B3n%20de%20VPN/Pasted_image_20250901184652.png)

Ahora vamos a crear una pareja de claves pública y privada para nuestro cliente Android ejecutando el siguiente comando:

```
wg genkey | tee movil_privada.key | wg pubkey | tee movil_publica.key.pub
```

![Pasted image 20250901184913.png](Instalaci%C3%B3n%20y%20configuraci%C3%B3n%20de%20VPN/Pasted_image_20250901184913.png)

Una vez, que hayamos generado las claves, vamos a ejecutar el siguiente comando para poder crear un archivo de configuración vacío para nuestro cliente android:

```
touch movil.conf
```

![Pasted image 20250901185140.png](Instalaci%C3%B3n%20y%20configuraci%C3%B3n%20de%20VPN/Pasted_image_20250901185140.png)

Copiamos la clave privada de nuestro cliente android dentro de este archivo de configuración que acabamos de crear ejecutando el siguiente comando:

```
cat movil_privada.key > movil.conf
```

![Pasted image 20250901185351.png](Instalaci%C3%B3n%20y%20configuraci%C3%B3n%20de%20VPN/Pasted_image_20250901185351.png)

Repetiremos el mismo proceso, pero esta vez con la clave pública de nuestro servidor:

```
cat ../../claves_servidor/servidor_publica.key >> movil.conf
```

![Pasted image 20250901185530.png](Instalaci%C3%B3n%20y%20configuraci%C3%B3n%20de%20VPN/Pasted_image_20250901185530.png)

Abrimos el archivo de configuración `movil.conf` con el editor nano:

```
nano movil.conf
```

![Pasted image 20250901190152.png](Instalaci%C3%B3n%20y%20configuraci%C3%B3n%20de%20VPN/Pasted_image_20250901190152.png)

Donde contendrá la siguiente configuración:

```
[Interface]
Address = 192.168.2.2
PrivateKey = <Clave Privada del cliente>
ListenPort = 51820

[Peer]
PublicKey = <Clave Pública del servidor>
Endpoint = IP_pública_del_servidor:51820
AllowedIPs = 0.0.0.0/0
```

![Pasted image 20250901190545.png](Instalaci%C3%B3n%20y%20configuraci%C3%B3n%20de%20VPN/Pasted_image_20250901190545.png)

Guardamos y cerramos el editor nano, ahora debemos de copiar nuestra clave pública de nuestro cliente android en nuestro archivo de configuración `wg0.conf`:

```
cat movil_publica.key.pub >> /etc/wireguard/wg0.conf
```

![Pasted image 20250901190957.png](Instalaci%C3%B3n%20y%20configuraci%C3%B3n%20de%20VPN/Pasted_image_20250901190957.png)

Abrimos el archivo de configuración `wg0.conf` con el editor nano:

```
nano wg0.conf
```

![Pasted image 20250901191056.png](Instalaci%C3%B3n%20y%20configuraci%C3%B3n%20de%20VPN/Pasted_image_20250901191056.png)

Donde vamos a agregar el siguiente bloque:

```
[Peer]
PublicKey = <Clave pública del cliente>
AllowedIPs = 192.168.2.2/32
PersistentKeepAlive = 25
```

Y se quedará de la siguiente manera:
Guardamos y cerramos el editor nano y vamos a ejecutar el siguiente comando para poder proceder con el reinicio de Wireguard:

![image.png](Instalaci%C3%B3n%20y%20configuraci%C3%B3n%20de%20VPN/image.png)

```
systemctl restart wg-quick@wg0
```

![Pasted image 20250901191728.png](Instalaci%C3%B3n%20y%20configuraci%C3%B3n%20de%20VPN/Pasted_image_20250901191728.png)

Para poder comprobar de que está corriendo el servicio, debemos de ejecutar el siguiente comando:

```
systemctl status wg-quick@wg0
```

![Pasted image 20250901191738.png](Instalaci%C3%B3n%20y%20configuraci%C3%B3n%20de%20VPN/Pasted_image_20250901191738.png)

Y para poder comprobar si Wireguard nos ha permitido la configuración, debemos de ejecutar el siguiente comando:

```
wg
```

![Pasted image 20250901192019.png](Instalaci%C3%B3n%20y%20configuraci%C3%B3n%20de%20VPN/Pasted_image_20250901192019.png)

Como hemos podido observar en la anterior captura nos ha dejado perfectamente, ahora vamos a instalar el siguiente programa para poder pasar nuestro archivo de configuración cliente android a nuestro dispositivo móvil:

```
apt install qrencode
```

![Pasted image 20250901192248.png](Instalaci%C3%B3n%20y%20configuraci%C3%B3n%20de%20VPN/Pasted_image_20250901192248.png)

Una vez, que lo tengamos instalado, vamos a generar el código QR con el siguiente comando:

```
qrencode -t ansiutf8 < movil.conf
```

![Pasted image 20250901192427.png](Instalaci%C3%B3n%20y%20configuraci%C3%B3n%20de%20VPN/Pasted_image_20250901192427.png)

Nos descargamos la aplicación desde el Play Store y desde la aplicación escaneamos el código QR.

## 3.2 Windows

Para poder configurar nuestro cliente Windows, debemos de descargarnos el programa Wireguard desde la página oficial [https://www.wireguard.com/install/](https://www.wireguard.com/install/)

Una vez, que lo tengamos descargado, debemos de añadir un túnel vacío:

![Pasted image 20250901195051.png](Instalaci%C3%B3n%20y%20configuraci%C3%B3n%20de%20VPN/Pasted_image_20250901195051.png)

Donde añadiremos la siguiente configuración:

![Pasted image 20250901195520.png](Instalaci%C3%B3n%20y%20configuraci%C3%B3n%20de%20VPN/Pasted_image_20250901195520.png)

Una vez realizado lo anterior, debemos de dirigirnos a nuestro servidor, nos dirigimos a la carpeta `clientes` que hemos creado anteriormente y vamos a crear una carpeta llamada `windows` con el siguiente comando:

```
mkdir windows
```

![Pasted image 20250901201824.png](Instalaci%C3%B3n%20y%20configuraci%C3%B3n%20de%20VPN/Pasted_image_20250901201824.png)

Y ahora vamos a editar el archivo de configuración `wgo.conf` con el editor nano

```
nano /etc/wireguard/wg0.conf
```

![Pasted image 20250901202722.png](Instalaci%C3%B3n%20y%20configuraci%C3%B3n%20de%20VPN/Pasted_image_20250901202722.png)

Donde vamos a agregar el siguiente bloque:

```
[Peer]
# Cliente Windows
PublicKey = <Clave pública del cliente>
AllowedIPs = 192.168.2.3/32
PersistentKeepAlive = 25
```

Guardamos y cerramos el editor nano y procedemos con el reinicio de Wireguard:

```
systemctl restart wg-quick@wg0
```

![Pasted image 20250901202722.png](Instalaci%C3%B3n%20y%20configuraci%C3%B3n%20de%20VPN/Pasted_image_20250901202722%201.png)