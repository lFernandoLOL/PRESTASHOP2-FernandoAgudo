# BALANCEADOR PRESTASHOP CON NFS: RECUPERACION IAW FERNANDO AGUDO CERRO 

# Índice
#### [1. Aprovisionamiento de las máquinas ](#id1)
###### - [1.1 Aprovisionamiento del balanceador ](#id2)
###### - [1.2 Aprovisionamiento de los nginx ](#id3)
###### - [1.3 Aprovisionamiento del NFS ](#id4)
###### - [1.4 Aprovisionamiento del SQL ](#id5)
###### - [1.5 VagrantFile ](#id6)
#### [2. Configuración de las máquinas ](#id7)
###### - [2.1 Configuración del SQL ](#id8)
###### - [2.2 Configuración del NFS ](#id9)
###### - [2.3 Configuración de los NGINX ](#id10)
###### - [2.4 Configuración del balanceador ](#id11)

## Aprovisionamiento<a name="id1"></a>

##### El aprovisionamiento del balanceador<a name="id2"></a>
![](/IMAGENES/AprovBal.png)

##### El aprovisionamiento de las máquinas nginx:<a name="id3"></a>
![](/IMAGENES/AprovNginx.png)

##### El aprovisionamiento de el servidor NFS:<a name="id4"></a>
![](/IMAGENES/AprovNFS.png)

##### El aprovisionamiento del SQL:<a name="id5"></a>
![](/IMAGENES/AprovSQL.png)

##### El VagrantFile: <a name="id6"></a>
![](/IMAGENES/vagrantfile.png)

## Configuración de las máquinas<a name="id7"></a>
##### Para conectarnos a nuestras máquinas, el primer paso es levantarlas y ponerlas en funcionamiento, para ello utilizamos el comando "vagrant up"
![](/IMAGENES/vagrantup.png)

![](/IMAGENES/vagrantstatus.png)

### Configuración del servidor SQL:<a name="id8"></a>

##### Para configurar sql, ponemos el comando mysql_secure_installation, 
![](/IMAGENES/sql/sql1.png)


##### Creamos la base de datos para prestashop y su usuario y contraseña que tendrán permiso sobre esta.
![](/IMAGENES/sql/sql2.png)
![](/IMAGENES/sql/sql3.png)


### Configuración del NFS:<a name="id9"></a>
![]()
###### Nos conectamos a la máquina con el comando "vagrant ssh FernandoAgudoNfs", y creamos la carpeta la cual vamos a compartir con los nginx, y también modificamos la propiedad de usuario y grupo para que los clientes puedan acceder sin restricciones:
![](/IMAGENES/nfs/nfs1.png)
![](/IMAGENES/nfs/nfs.png)

###### Lo siguiente que haremos, es ir a el archivo "exports" que se crea al instalar el servicio nfs-server. Aquí, indicaremos las ip de nuestros nginx que se van a conectar, seguidos de parámetros rw, sync, no_subtree_check
![](/IMAGENES/nfs/nfs2.png)


###### El paso siguiente será configurar el php, para que funcione en modo socket. Para ello, debemos poner el puerto en el que vamos a escuchar en el archivo www.conf de php (/etc/php/7.4/fpm/pool.d/www.conf):
![](/IMAGENES/nfs/nfs4.png)

###### Reiniciamos el servicio php, y si no tenemos ningún error podremos continuar
![](/IMAGENES/nfs/nfs5.png)


###### Nos dirigimos a la carpeta que compartimos del NFS, en este caso /var/www/prestashop, e instalamos el cms y lo descomprimimos, dejando nuestro CMS listo.
![](/IMAGENES/nfs/nfs6.png)

###### Ahora nos queda atacar a la base de datos, lo cual lo conseguimos editando el siguiente archivo de prestashop:
![](/IMAGENES/nfs/dbslave1.png)
###### En el, añadimos los datos de la base de datos del servidor sql:
![](/IMAGENES/nfs/dbslave2.png)


### Configuración de los Nginx:<a name="id10"></a>

###### Nos conectamos a la máquina con el comando "vagrant ssh FernandoAgudoNginx", y creamos la carpeta donde vamos a albergar los archivos compartidos del servidor NFS:
![](/IMAGENES/nginx/nginx1.png)

###### Montamos la carpeta mediante el comando mount seguido de la ip del servidor NFS.
###### Luego, añadimos la carpeta compartida a fstab para que se quede permanente:
![](/IMAGENES/nginx/nginx2.png)
![](/IMAGENES/nginx/fstab1.png)
![](/IMAGENES/nginx/fstab2.png)

###### Configuramos el servicio de Nginx: Vamos a la ruta /etc/nginx/sites-available/ y editamos el archivo default, poniendo la ruta de nuestra carpeta con el CMS, y añadiendo "index.php", ya que este cms utiliza php.
![](/IMAGENES/nginx/nginx3.png)


###### Además, como vamos a trabajar por socket, debemos incluir las lineas de fastcgi seguido de la ip del servidor nfs y del puerto en el que trabaja php
![](/IMAGENES/nginx/nginx4.png)

###### Tras esto, creamos el enlace a sites-enabled y reiniciamos el servicio de nginx con el comando "systemctl restart nginx". Y repetimnos todo el proceso de nginx en nuestra máquina de nginx2.

![](/IMAGENES/nginx/nginx5.png)


### Configuración del Balanceador<a name="id11"></a>

#### Modo no seguro
###### Añadimos las ip a balancear
![](/IMAGENES/bal/bal1.png)
![](/IMAGENES/bal/bal2.png)
###### Creamos el enlace y reiniciamos el servicio, habiendo previamente borrado el default
![](/IMAGENES/bal/bal3.png)

###### Nos conectamos por su ip pública
![](/IMAGENES/bal/bal4.png)

#### Modo seguro:
###### Creamos las claves certificadoras mediante el comando:
![](/IMAGENES/bal/seguro1.png)
###### Añadimos la ruta de las claves, y omitimos la verificación, puesto que no son claves certificadoras reales
![](/IMAGENES/bal/seguro2.png)
###### Accedemos, pero ahora desde HTTPS
![](/IMAGENES/bal/seguro3.png)