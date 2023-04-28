# Plex sobre Docker en Orange Pi

Pueden encontrar el repositorio original dedicado a raspberry pi [aqui](https://github.com/pablokbs/plex-rpi)

Con este repo puedes crear tu propio server que descarga tus series y peliculas automáticamente usando flexget y trnasmission, y cuando finaliza, las copia al directorio `media/` donde Plex las encuentra y las agrega a tu biblioteca.

## Requerimientos iniciales

Agregar tu usuario (cambiar `kbs` con tu nombre de usuario)

```
sudo useradd kbs -G sudo
```

Agregar esto al sudoers para correr sudo sin password 

Nota: puedes acceder a la configuracion mencionada con el comando: `sudo visudo`
```
%sudo   ALL=(ALL:ALL) NOPASSWD:ALL
```

Agregar esta linea a `etc/ssh/sshd_config` para que sólo tu usuario pueda hacer ssh

Nota: cambiar `kbs` con tu usuario

```
echo "AllowUsers kbs" | sudo tee -a /etc/ssh/sshd_config
sudo systemctl enable ssh && sudo systemctl start ssh
```

Instalar paquetes básicos

```
sudo apt-get update && sudo apt-get install -y \
     apt-transport-https \
     ca-certificates \
     curl \
     gnupg2 \
     software-properties-common \
     vim \
     fail2ban \
     ntfs-3g
```

## Instalar Docker 

Para esto aprovecharemos las funcionalidades gráficas de armbian

```
sudo armbian-config
```


Nos dirigimos a la opción Software y presionamos Enter
![hola](img/armbian-config-1.png)


Damos Enter sobre _Softy_

![hola](img/armbian-config-2.png)


Buscamos la opción Docker y lo seleccionamos presionando la barra espaciadora, posterior presionamos enter para que comience la instalación. Una vez que finalice podemos salir del asistente de configuración.


![hola](img/armbian-config-3.png)


Instala docker-compose

```
sudo apt install docker-compose
```


Agregar tu usuario al grupo docker 

Nota cambiar `kbs` por tu usuario
```
# Add kbs to docker group
sudo usermod -a -G docker kbs
#(logout and login)
```

## Montar disco externo

Montar el disco (es necesario ntfs-3g si es que tienes tu disco en NTFS)
NOTA: en este [link](https://youtu.be/OYAnrmbpHeQ?t=5543) pueden ver la explicación en vivo

```
# usamos la terminal como root porque vamos a ejecutar algunos comandos que necesitan ese modo de ejecución
sudo su
# buscamos el disco que querramos montar (por ejemplo la partición sdb1 del disco sdb)
fdisk -l
# pueden usar el siguiente comando para obtener el UUID
ls -l /dev/disk/by-uuid/
# y simplemente montamos el disco en el archivo /etc/fstab (pueden hacerlo por el editor que les guste o por consola)
echo UUID="{nombre del disco o UUID que es único por cada disco}" {directorio donde queremos montarlo, por ejemplo /mnt/storage} ntfs-3g defaults,auto 0 0 | \
     sudo tee -a /etc/fstab
# por último para que lea el archivo fstab
mount -a (o reiniciar)
```

Modifica tu docker config para que guarde los temps en el disco:

```
sudo vim /etc/default/docker
# Agregar esta linea al final con la ruta de tu disco externo montado
export DOCKER_TMPDIR="/mnt/storage/docker-tmp"
```

## Cómo correrlo

Simplemente bajate este repo y modificá las rutas de tus archivos en el archivo (oculto) .env, y después corré:

`docker-compose up -d`

## IMPORTANTE

Las raspberry son computadoras excelentes pero no muy potentes, y plex por defecto intenta transcodear los videos para ahorrar ancho de banda (en mi opinión, una HORRIBLE idea), y la chiquita raspberry no se aguanta este transcodeo "al vuelo", entonces hay que configurar los CLIENTES de plex (si, hay que hacerlo en cada cliente) para que intente reproducir el video en la máxima calidad posible, evitando transcodear y pasando el video derecho a tu tele o Chromecast sin procesar nada, de esta forma, yo he tenido 3 reproducciones concurrentes sin ningún problema. En android y iphone las opciones son muy similares, dejo un screenshot de Android acá:

<img src="https://i.imgur.com/F3kZ9Vh.png" alt="plex" width="400"/>

Más info acá: https://github.com/pablokbs/plex-rpi/issues/3
