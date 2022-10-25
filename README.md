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
Por ejemplo, se pueden cambiar a los de google:
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

## Configuración de zona
El fichero que se va a usar para configurar nuestra zona es

