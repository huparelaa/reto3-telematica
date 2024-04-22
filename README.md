# ST0263 - Tópicos Especiales de Telemática
#
## Integrantes:
- Hobarlan Uparela Arroyo (huparelaa@eafit.edu.co)
- Julian David Valencia Restrepo (jdvalencir@eafit.edu.co)
- Andres Prada Rodriguez (apradar@eafit.edu.co)

## Profesor
- **Nombre:** Edwin Nelson Montoya Munera
- **Correo:** emontoya@eafit.edu.co

# Proyecto 1 (Sistema de archivos distribuidos)

## 1. breve descripción de la actividad

En este proyecto, desplegaremos una aplicación monolítica usando WordPress en un entorno Docker, con un enfoque en alta disponibilidad y escalabilidad. Utilizaremos Nginx como balanceador de carga para distribuir el tráfico entre dos instancias de WordPress, optimizando así el rendimiento y la fiabilidad. Además, la arquitectura incluirá instancias separadas para la base de datos MySQL y el almacenamiento de archivos mediante NFS, todo alojado en AWS con cinco máquinas virtuales diseñadas para asegurar un funcionamiento continuo y eficiente, además de tener una instancia adicional (bastion host) para acceder a las instancias privadas (Wordpress, base de datos y NFSServer). Este esquema no solo mejora la gestión y el mantenimiento del sistema, sino que también refuerza la seguridad y la capacidad de respuesta frente a altas demandas de usuario.

### 1.1. Que aspectos cumplió o desarrolló de la actividad propuesta por el profesor (requerimientos funcionales y no funcionales)

Aspectos cumplidos de la actividad propuesta por el profesor:

* Aplicación wordpress o app seleccionada dockerizada monolítica en varios nodos que mejore la
disponibilidad de esta aplicación.
* Implementar un balanceador de cargas basado en nginx que reciba el tráfico web https de
Internet con múltiples instancias de procesamiento.
* Tener 2 instancias de procesamiento wordpress o app seleccionada detrás del balanceador de
cargas.
* Tener 1 instancia de bases de datos mysql.
* Tener 1 instancia de archivos distribuidos en NFS.
* Arquitectura propuesta en el documento guía implementada.
* Despliegue correcto con dominio y subdominio propio.

### 1.2. Que aspectos NO cumplió o desarrolló de la actividad propuesta por el profesor (requerimientos funcionales y no funcionales)

* Ninguno

## 2. información general de diseño de alto nivel, arquitectura, patrones, mejores prácticas utilizadas.
Se utilizó la arquitectura presentada a continuación:
![img](/imgs/overview.png)

Como podemos ver en el diagrama se puede acceder a la página web por medio de `https` debido al certificado SSL obtenido, además como se puede apreciar esta dirección de internet va al balanceador de cargas el cual se encarga de distribuir las peticiones entre una instancia de wordpress y otra, finalmente vemos que ambas instancias de wordpress apuntan a la misma base de datos y NFS Server los cuales también están separadas en sus respectivas máquinas virtuales usando docker tal y como se evidencia en la imagen.
## 3. Descripción del ambiente de desarrollo y técnico: lenguaje de programación, librerias, paquetes, etc, con sus numeros de versiones.

Se utilizó Docker, con sus respectivos docker-compose para contenerizar correctamente las aplicaciones y servicios necesarios, puede encontrar estos archivos en la carpeta `docker` de este repositorio.

## 4. Descripción del ambiente de EJECUCIÓN (en producción) lenguaje de programación, librerias, paquetes, etc, con sus numeros de versiones.



## IP o nombres de dominio en nube o en la máquina servidor.

Para acceder a la página web se puede hacer por medio de la siguiente dirección: https://reto3.teleafit.tech/

* **Balanceador de carga:** IP elástica de AWS: 3.215.163.106.

* **Wordpress 1:** IP privada: 10.0.0.37

* **Wordpress 2:** IP privada: 10.0.0.40

* **Base de datos:** IP privada:10.0.0.42

* **NFS Server:** IP privada: 10.0.0.46

* **Bastion Host:** IP pública: 54.81.76.221

![instancias](/imgs/instancias.png)


## Descripción y como se configura los parámetros del proyecto

Para configurar el proyecto se deben seguir los siguientes pasos:

### Crear instancia de bastion host

1. Crear una instancia de bastion host en AWS.
2. Crear un par de llaves para acceder a la instancia.
3. Configurar el grupo de seguridad para permitir el acceso a la instancia de bastion host.
4. Acceder a la instancia de bastion host por medio de ssh.

### Crear instancia de balanceador de carga usando Nginx

1. Copia el archivo `docker-compose.yml` y `nginx.conf` en una instancia de balanceador de carga.

2. Configura el archivo `nginx.conf` con la siguiente información:

```nginx
http {
    upstream backend {
        server 10.0.0.37;
        server 10.0.0.40;
    }
}
```

Reemplaza las direcciones IP por las direcciones IP de las instancias de wordpress.

3. Ejecuta el siguiente comando para levantar el servicio de Nginx:

```bash
docker-compose up -d
```

### Crear instancia de Wordpress

1. Copia el archivo `docker-compose.yml` en una instancia de Wordpress.

2. Configura el archivo `docker-compose.yml` con la siguiente información:

```yaml
version: '3'

services:
  wordpress:
    container_name: wordpress
    image: wordpress
    ports:
      - 80:80
    restart: always
    environment:
        WORDPRESS_DB_HOST:
        WORDPRESS_DB_USER:
        WORDPRESS_DB_PASSWORD:
        WORDPRESS_DB_NAME:
    volumes:
        - /home/ubuntu/wordpress:/var/www/html
```

Reemplaza las variables de entorno por las credenciales de la base de datos.

3. Ejecuta el siguiente comando para levantar el servicio de Wordpress:

```bash
docker-compose up -d
```

### Crear instancia de base de datos

1. Copia el archivo `docker-compose.yml` en una instancia de base de datos.

2. Configura el archivo `docker-compose.yml` con la siguiente información:

```yaml
version: '3'
services:
  db:
    image: mariadb
    environment:
      MYSQL_ROOT_PASSWORD:
      MYSQL_DATABASE: 
      MYSQL_USER: 
      MYSQL_PASSWORD:
    volumes:
      - data:/var/lib/mysql
    ports:
      - "3306:3306"
volumes:
  data:
```

Reemplaza las variables de entorno por las credenciales de la base de datos.

3. Ejecuta el siguiente comando para levantar el servicio de la base de datos:

```bash
docker-compose up -d
```

### Crear instancia de NFS Server

1. **Instalación de NFS Server**

```bash
sudo apt-get update
sudo apt-get install nfs-kernel-server
```

2. **Configuración de NFS Server**

```bash
sudo mkdir -p var/nfs/worpress
sudo chown nobody:nogroup var/nfs/worpress
sudo chmod 777 var/nfs/worpress
```

3. **Exportación del directorio**

- Añade el directorio a `/etc/exports` con las configuraciones adecuadas para los clientes:
    ```
    /var/nfs/wordpress <IP_clientes>(rw,sync,no_subtree_check)
    ```
- Aplica los cambios: `sudo exportfs -a`.
- Reinicia el servidor NFS: `sudo systemctl restart nfs-kernel-server`.

### Configuración del Cliente NFS en Instancias de WordPress

1. **Instalación del Cliente NFS**:
   - `sudo apt install nfs-common`.
   
2. **Montaje del Directorio NFS**:
- Monta el directorio NFS en el punto de montaje deseado:
    ```
    sudo mkdir -p /var/www/html
    sudo mount <IP_servidor_NFS>:/var/nfs/wordpress /var/www/html
    ```
- Añade la configuración al `/etc/fstab` para montajes automáticos:
    ```
    <IP_servidor_NFS>:/var/nfs/wordpress /var/www/html nfs auto,noatime,nolock,bg,nfsvers=3,intr,tcp,actimeo=1800 0 0
    ```

Esta configuración asegura que las instancias de WordPress puedan compartir de manera eficiente archivos y datos a través del servidor NFS, manteniendo la consistencia entre múltiples servidores para una aplicación escalable y de alta disponibilidad.

## Resultados o pantallazos 

Video que muestra como se ejecuta el software: [video](https://youtu.be/iGte68D_GbQ)
[![Texto alternativo](https://img.youtube.com/vi/iGte68D_GbQ/maxresdefault.jpg)](https://www.youtube.com/watch?v=iGte68D_GbQ)


### Referencias
- [Wordpress with NFS](https://mohsensy.github.io/sysadmin/2020/04/01/wordpress-nfs.html)

- [How To Install TLS/SSL on Docker Nginx Container With Let’s Encrypt](https://macdonaldchika.medium.com/how-to-install-tls-ssl-on-docker-nginx-container-with-lets-encrypt-5bd3bad1fd48)