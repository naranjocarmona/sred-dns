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



## Configuracion DNS-DHCP
El servidor DHCP se va a instalar en la misma máquina que el dns primario.

Modificamos el archivo de configuracion
```
/etc/dhcp/dhcpd.conf
``` 

![](/capturas/configracionDNS-DHCP1.png)

Añadimos el nombre de dominio de nuestro servidor en nuestra máquina servidor.
Incluimos las ips del dns primario y del dns secundario.
También añadimos la ip del dns primario como option-routers.

Añadimos una subnet DHCP a nuestra máquina servidor

![](/capturas/configuracionDNS-DHCP2.png)


En la máquina dns primaria y secundaria añadimos al archivo de configuración los dns primario y secundario.
```
/etc/resolv.conf
```
![](/capturas/configuracionDNS-DHCP5.png)



## Comprobación

Abrimos una máquina cliente y vemos la configuración de red.


![](/capturas/configuracionDNS-DHCP3.png)
![](/capturas/configuracionDNS-DHCP4.png)

Vemos, que DHCP ha funcionado y recibe el nombre de dominio del servidor dns.
