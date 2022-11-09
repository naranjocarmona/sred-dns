- [Tarea 2. Fecha: 09/11/2022](#tarea-2-fecha--09-11-2022)
- [Información sobre MV y versión de SO](#informaci-n-sobre-mv-y-versi-n-de-so)
  * [VirtualBox](#virtualbox)
  * [Debian](#debian)
  * [Windows XP](#windows-xp)
- [Instalación de un servidor DNS](#instalaci-n-de-un-servidor-dns)
  * [Comprobación del estado del servicio](#comprobaci-n-del-estado-del-servicio)
  * [Comprobación de que el servicio dns funciona en nuestro servidor (localhost)](#comprobaci-n-de-que-el-servicio-dns-funciona-en-nuestro-servidor--localhost-)
- [Configuración del servidor DNS](#configuraci-n-del-servidor-dns)
  * [Diferentes archivos de configuración](#diferentes-archivos-de-configuraci-n)
  * [Ficheros de configuración en otras rutas](#ficheros-de-configuraci-n-en-otras-rutas)
  * [Fichero de configuración de zona](#fichero-de-configuraci-n-de-zona)
  * [Fichero de ruta de directorio de Bind](#fichero-de-ruta-de-directorio-de-bind)
  * [Configuración de servidor maestro](#configuraci-n-de-servidor-maestro)
  * [Comprobación](#comprobaci-n)
- [Instalación de un servidor DNS secundario](#instalaci-n-de-un-servidor-dns-secundario)
  * [Configuración del DNS secundario](#configuraci-n-del-dns-secundario)
  * [Modificaciones en la configuración del servidor DNS primario](#modificaciones-en-la-configuraci-n-del-servidor-dns-primario)
  * [Incluir el servidor DNS secundario en las configuraciones directa e inversa del primario](#incluir-el-servidor-dns-secundario-en-las-configuraciones-directa-e-inversa-del-primario)
  * [Comprobaciones desde el servidor DNS secundario](#comprobaciones-desde-el-servidor-dns-secundario)
- [Configuracion DNS-DHCP](#configuracion-dns-dhcp)
  * [Configuración DNS](#configuraci-n-dns)
  * [Configuración DHCP](#configuraci-n-dhcp)
  * [Comprobaciones desde el cliente (Windows XP)](#comprobaciones-desde-el-cliente--windows-xp-)


# Tarea 2. Fecha: 09/11/2022

# Información sobre MV y versión de SO
## VirtualBox
![](/capturas/info-virtualbox.png)
## Debian
![](/capturas/info-debian.png)
## Windows XP
![](/capturas/info-xp.png)

# Instalación de un servidor DNS
Instalar bind9 con el siguiente comando

```
apt install bind9
```

![instalacion de bind9](/capturas/instalacionBind9.png)

## Comprobación del estado del servicio

```
systemctl status bind9
```
![](/capturas/estadoServicio.png)


## Comprobación de que el servicio dns funciona en nuestro servidor (localhost)
![](/capturas/dig.png)

> Nota: Los servidores DNS por defecto del sistema se guardan en el siguiente fichero
```
/etc/resolv.conf
```
![](/capturas/dnsGoogle.png)



# Configuración del servidor DNS

## Diferentes archivos de configuración
La configuración del servidor se distribuye en diferentes ficheros en la carpeta `/etc/bind`:

- **named.conf.options**: En este fichero se encuentra la instrucción options que es donde se
incluyen todos los parámetros a nivel global de BIND.

- **named.conf.local**: Aquí es donde crearemos nuestras zonas con la instrucción zone.

- **named.conf.default-zones**: Este fichero contiene las instrucciones zone de algunas zonas que
incluye BIND por defecto.

- **db.local**: permite resolver el nombre de localhost.

## Ficheros de configuración en otras rutas

- **/usr/share/dns/root.hinst**: están guardados todos los servidores raiz.

- **/etc/hosts**: En este fichero se pueden establecer relaciones IP dominio manuales.

## Fichero de configuración de zona
El fichero que se va a usar para configurar nuestra zona es

```
/etc/bind/db.empty
```

![](/capturas/ficheroZona.png)


Este el fichero donde se guardan todas las zonas

```
/etc/bind/named.conf.default-zones
```

![](/capturas/ficheroGuardadoZonas.png)

## Fichero de ruta de directorio de Bind

Este fichero contiene la ruta del directorio bind
```
/var/cache/bind 
```
Es utilizado por BIND como referencia de todas las rutas relativas de ficheros.

![](/capturas/rutasRelativasFichero.png)

Vemos que tiene permisos de escritura y lectura

![](/capturas/permisosRutasRelativas.png)

## Configuración de servidor maestro

Lo primero que hacemos es configurar el fichero 

```
/etc/resolv.conf
```
Añadimos la línea nameservers y la ip del servidor, el nombre de dominio y la búsqueda.

![](/capturas/resolv.png)

Acontinuación accedemos a la carpeta

```
cd /var/cache/bind
```
Hacemos una copia del fichero de configuracion

```
db.local
```

Con la copia de este fichero creamos otro para la configuracion no inversa de DNS. Le ponemos el nombre de 
```
db.master.antonio.org
```

> Nota: le ponemos el nombre de dominio al fichero.

Configuramos el fichero:

![](/capturas/configuracionMasterNoInverso.png)

Añadimos el origen (el dominio), el nombre del servidor y sus ips.

> Nota: Es muy importante hacerlo como en la imagen. No olvidar los espacios y los puntos (.)correspondientes a poner.

Volvemos a hacer otra copia del fichero db.local, pero esta vez para configurar e DNS inverso

```
db.master.192.168.0.rev
```
> Nota: añadimos la ip del servidor sin el ultimo numero.

Configuramos el fichero

![](/capturas/configuracionMaterInverso.png)

Configuramos origen tal como en la imagen y especificamos el nombre del servidor.



Seguidamente nos vamos a 

```
/etc/bind/named.conf.options
```

![](/capturas/configuracionNamedConfOptions.png)

Aquí añadimos que el dns escuché a través del puerto 53 y volvemos a dejar el fichero como en la imagnen.

Finalmente declaramos las zonas, dentro de 

```
/etc/bind/named.conf.local
```

![](/capturas/configuracionNamedConfLocal.png)


Añadimos zona para la configuración no inversa y para la inversa como se muestra en la imagen.


## Comprobación

Para comprobar que todo funciona correctamente, lo comprobamos de la siguiente manera:

Para la configuracion no inversa:

![](/capturas/checkzzoneNoInverso.png)

Para la configuracion inversa:

![](/capturas/checkzoneInverso.png.png)

Comprobamos el estado de bind9

![](/capturas/bind9Activo.png)

Hacemos ping al servidor y clientes

![](/capturas/pingServidor.png)

Hacemos dig al dominio

![](/capturas/digAntonio.png)


> Nota: para hacer estas comprobaciones, es obligatorio hacerlo desde la carpeta donde están ubicados los ficheros. La carpeta sería

```
/var/cache/bind/

```

# Instalación de un servidor DNS secundario
Instalar en otra máquina bind9 con el siguiente comando

```
apt install bind9
```

## Configuración del DNS secundario
Cambiamos la IP y la configuración DNS de la siguiente manera:
![](/capturas/dnsSecundarioConfZonas.png)

Cambiamos el tipo de `master` a `slave`, y añadimos la cláusula "masters" y entre llaves, ponemos cuál es la IP del servidor DNS primario.

## Modificaciones en la configuración del servidor DNS primario
Hay que decirle al servidor DNS primario que permita las conexiones al secundario.
Modificamos el fichero `named.conf.local` y añadimos `allow-transfer` y `also-notify`:

![](/capturas/permitirDnsSecundario.png)

## Incluir el servidor DNS secundario en las configuraciones directa e inversa del primario
Directa:
![](/capturas/confDirectaDnsPrimarioSecundario.png)
Inversa:
![](/capturas/confInversaDnsPrimarioSecundario.png)

## Comprobaciones desde el servidor DNS secundario
Ejecutar el siguiente comando:
```
dig servidor.antonio.org
```
![](/capturas/comprobacionDigSecundario.png)

Ejecutar el siguiente comando:
```
dig @192.168.0.220 servidor.antonio.org
```
![](/capturas/comprobacionSecundarioInversa.png)

Se pueden hacer más comprobaciones con el comando `nslookup`:
![](/capturas/secundarioNSlookup.png)

Es conveniente añadir la IP del servidor DNS secundario al fichero resolv:
![](/capturas/secundarioResolv.png)



# Configuracion DNS-DHCP
Para esta práctica se crean 3 máquinas virtuales nuevas. Dos debian, que van a ser el servidor DHCP y el DNS, respectivamente, y un Windows XP que hará de cliente.

## Configuración DNS
Se procede a instalar `bind9` como se explica previamente en esta guía. 
Principalmente se modifican los siguientes ficheros de configuración:

* /etc/bind/named.conf.options
```
options {
    directory "/var/cache/bind";
    listen-on port 53 { any; };
    recursion yes;
    allow-recursion { localnets; };
};
```
![](/capturas/dns-dhcp-conf-options.png)

* /etc/bind/named.conf.local
```
zone "antonio.org" in {
    type master;
    file "db.master.antonio.org";
};

zone "0.168.192.IN-ADDR.ARPA" in {
    type master;
    file "db.master.192.168.0.rev";
};
```
![](/capturas/dns-dhcp-named-local.png)

* /var/cache/bind/db.master.antonio.org
```                                                                                           
;
; BIND data file for local loopback interface
;
$TTL    604800
$ORIGIN antonio.org.
@       IN      SOA     dns.antonio.org. root.antonio.org. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      dns.antonio.org.
dns     IN      A       192.168.0.198
```
![](/capturas/dns-dhcp-antonio.png)

* /var/cache/bind/db.master.192.168.0.rev
```
;
; BIND data file for local loopback interface
;
$TTL    604800
$ORIGIN 0.168.192.IN-ADDR.ARPA.
@       IN      SOA     dns.antonio.org. root.antonio.org. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      dns.antonio.org.
198     IN      PTR     dns.antonio.org.
```
![](/capturas/dns-dhcp-antonio-rev.png)


## Configuración DHCP
Se procede a instalar `isc-dhcp-server` como se explica en [esta guía](https://github.com/naranjocarmona/sred-dhcp). 
Principalmente se modifica el siguiente fichero de configuración:

* /etc/dhcp/dhcpd.conf
```
option domain-name "dhcp.local";
option domain-name-servers 8.8.8.8;
option routers 192.168.0.1;
default-lease-time 86400;
max-lease-time 691200;
min-lease-time 7200;

authoritative;

allow-unknow-clients;
ddns-updates on;
ddns-update-style interim;
ddns-domainname "antonio.org";
ddns-rev-domainname "in-addr.arpa.";

subnet 192.168.0.0 netmask 255.255.255.0 {
    range 192.168.0.200 192.168.0.240;
    option domain-name "dns.antonio.org";
    option domain-name-servers 192.168.0.198;
    option routers 192.168.0.120;
    option broadcast-address 192.168.0.255;
    default-lease-time 600;
    max-lease-time 7200;
}

host dns.antonio.org {
  hardware ethernet 08:00:27:d8:0b:98;
  fixed-address 192.168.0.198;
}
```

Con las opciones `ddns-updates` y `ddns-update-style interim` activamos la actualización automática al DNS.

Definimos DHCP para que dé IPs en el rango de 192.168.0.200 a 192.168.0.240.

Asignamos la IP fija 192.168.0.198 al servidor DNS a través de `fixed-address`.

## Comprobaciones desde el cliente (Windows XP)
Al iniciar la máquina en la misma red, ésta debe encontrar los servidores DNS y DHCP configurados, así como obtener una IP por DHCP del rango definido.
![](/capturas/dns-dhcp-cliente.png)


# Configurar DHCP y DNS con DNSMASQ

Asignamos una IP fija, modificando el fichero de interfaces:
```
nano /etc/network/interfaces
```
Por ejemplo:
```
source /etc/network/interfaces.d/*

auto lo
iface lo inet loopback

auto enp0s3
iface enp0s3 inet static
address 192.168.0.123
netmask 255.255.255.0
gateway 192.168.0.1
```

Instalamos las herramientas necesarias:
```
apt install dnsmasq dnsmasq-base dnsmasq-utils
```

Comprobamos si se está ejecutando con:
```
systemctl status dnsmasq.service
```
![](/capturas/dnsmasq-estado-servicio.png)

Por seguridad, hacemos una copia de seguridad del fichero de configuración de DNSMASQ:
```
cp /etc/dnsmasq.conf /etc/dnsmasq.conf.ORIGINAL
```

Procedemos a cambiar el fichero `/etc/dnsmasq.conf`:

* Cambiamos el puerto a 53 en `port`
* Descomentamos `domain-needed`
* Descomentamos `borgus-priv`
* Configuramos las interfaces de red en `interface`
* Configuramos las IPs en `listen-address`
* Descomentamos `bind-interfaces`
* Descomentamos `expand-hosts`
* Configuramos el dominio en `domain`
* Configuramos el rango de IPs en `dhcp-range`. Se define el rango de IPs, junto con la máscara de red y el tiempo de liberación por defecto.
* Configuramos el tiempo de liberación máxima con `dhcp-lease-max`
* Descomentamos `dhcp-leasefile`
* Descomentamos `dhcp-authoritative`
* Descomentamos `log-dhcp` para que se escriban logs en el sistema.
* Descomentamos `conf-dir`

El fichero quedaría así:
```
port=53
domain-needed
bogus-priv
interface=lo,enp0s3
listen-address=127.0.0.1,192.168.0.123
bind-interfaces
expand-hosts
domain=antonio.org
dhcp-range=192.168.0.210,192.168.0.240,255.255.255.0,2h
dhcp-lease-max=86400
dhcp-leasefile=/var/lib/misc/dnsmasq.leases
dhcp-authoritative
log-dhcp
conf-dir=/etc/dnsmasq.d
```

Ahora, reiniciamos el servicio para que los cambios se apliquen.
```
systemctl restart dnsmasq.service
```

Viendo el estado, se comprueba algunos de los cambios, como el puerto 53 y el rango de IPs definido.
![](/capturas/dnsmasq-despues-de-configurar.png)

Por último, comprobamos con un Windows XP como máquina cliente y vemos que le asigna una IP del rango y contiene los parámetros del servidor que acabamos de configurar:
![](/capturas/dnsmasq-cliente.png)

Comprobando los logs del sistema, también se verifica que el servidor está funcionando correctamente:
![](/capturas/dnsmasq-logs.png)
