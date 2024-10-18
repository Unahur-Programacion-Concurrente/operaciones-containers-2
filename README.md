# operaciones-containers-2

## Imágenes y Contenedores (Ejercicio de Repaso) 
        
1. Creación de un contenedor Ubuntu (sistema operativo base en contenedor) 

   1.1. Buscar la Imagen  
      La imagen se puede buscar en  [https://hub.docker.com](https://hub.docker.com/search?q=ubuntu&type=image)  
      o directamente por comando:  
      ```
      docker search ubuntu
      ```

   1.2. Descargar la imagen (opcional)  
      ```
      docker pull ubuntu
      ```
      Si no se especifica la versión, descarga la última:  
      ```ubuntu:latest```

   1.3. Para ver las imagenes descargadas en nuestra máquina local  
     ```
     docker images 
     ```

   1.4. Crear contenedor ubuntu 
    ``` 
    docker run -it ubuntu
    ```  
    Al terminar queda en el shell del contenedor
    
    ```
    root@e2138ee6f5f9:/# 
    ```
    
    1.5 Salir del contenedor (deteniendolo)
    ```
    exit
    ```
    1.6 Verificar que el contenedor está detenido
    ```
    docker ps -a
    ```
    (Anotar el id del contenedor)
 
    1.7. Volver a arrancar el contenedor detenido
    ```
    docker start <id>  # o primeras letras del id
    ```
    1.8. Salir del shell del contenedor sin detenerlo

    ``` Ctrl+P+Q ```

    1.9. Para conectarse al shell de un contenedor en ejecución 
    ``` 
    docker attach <id>  # o primera letra del id  
    ```

2. Instalar mysql server en el contenedor ubuntu  
    2.1. Ingresar al contenedor Ubuntu
    ```
    sudo docker ps - a  # anotar el id del contenedor  
    sudo docker start <id>  
    sudo docker attach <id>
    ```
    2.2. Instalación de mysql 
    ``` 
    apt update  
    apt install mysql-server  
    ```
    2.3. Arrancar mysql 
    ``` 
    service mysql start  
    service mysql status
    ```
    2.4 Configurar arranque del servicio mysql en startup
    ```
    echo "service mysql start" >> ~/.bashrc
    ```
    2.5 Salir del contenedor

    ```Ctrol+P+Q```

    o (detiene el contenedor)
    ```
    exit 
    ```

3. Crear una imagen a partir del contenedor

    3.1 Comando a utilizar:
    ``` docker commit <container_id> <image_name>```

    3.2 Crear la imagen
    ```
    docker ps # anotar el id del contenedor
    docker commit  id_contenedor mimysql
    ```

    3.3 Listar las imagenes
    ```
    docker images
    ```

    3.4 Crear un contenedor utilizando la imagen nueva
    ```
    docker run -it --name midb mimysql
    ```
    3.5 Verificar el contenedor nuevo
    ```
    docker ps -a
    ```

## Comandos para eliminar contenedores e imágenes

1. Listar los contenedores en ejecución
    ```
    docker ps
    ```

2. Detener los contenedores en ejecución  
    ```
    docker stop <id> 
    ```

3. Listar todos los contenedores  
    ```
    docker ps -a
    ```

4. Eliminar todos los contenedores detenidos (stopped) 
    * Eliminar uno o mas:
        ```
        docker rm <id1> <id2>...
        ``` 
    
    * Eliminar todos (stopped):
        ```
        docker container prune
        ```
    
5. Listar las imágenes
    ```
    docker images
    ```
            
6. Borrar todas la imágenes no utilizadas  
    * Eliminar una o mas  
    ```
    docker rmi <id1>..
    ```
    * Eliminar todas (no utilizadas)
    ```
    docker image prune -a
    ```

## Crear contenedor MySQL desde Imagen Oficial (ejercicio)

1. Creación de un contenedor MySQL Server  
    1.1. Entrar a docker hub y buscar la imagen oficial de MySQL y buscar las instrucciones para arrancar contenedores.

    1.2. Arrancar un contenedor MySQL
    ```
    docker run --name mysql-server -e MYSQL_ROOT_PASSWORD=1234 -d mysql
    
    ```

    1.3. Listar las imágenes y contenedores, comprobar que el contenedor 
    mysql-server esta activo
    ```  
    docker ps  
    docker images
    ```

## Conectividad de Contenedores (ejercicio)

1. Conectividad entre contenedores (red interna de docker)
    1.1. Conectarse desde un contenedor cliente mysql (utilizando la red interna por defecto)
    * Obtener la dirección IP interna del contenedor mysql
        ```
        docker container inspect mysql-server
        ```
        Buscar la sección "Networks" - "bridge" - "IPAddress" y anotarla.

    * Conectarse con un contenedor (tarea) cliente mysql
    ```
    docker run -it --rm mysql mysql -h<IPAddress> -uroot -p
    ```

    1.2 Conectividad con DNS interno de docker utilizando los nombres de container en lugar de direcciones IP

    * Crear un network
    ```
    docker network create mired
    ```
    * Conectar el contenedor a la red
    ```
    docker network connect mired mysql-server
    ```

    Nota: otra alternativa es volver a crear el contenedor conectandolo a la red en docker run

    ```
    docker run --network mired --name mysql-server -e MYSQL_ROOT_PASSWORD=1234 -d mysql
    ```
    * Conectarse con un contenedor (tarea) cliente mysql utilizando la red y el nombre del contenedor en lugar de su dirección IP
    ```
    docker run -it --rm --network mired mysql mysql -hmysql-server -uroot -p
    ```
2. Conectividad de contenedores con el exterior (HOST)

5. Eliminar el contenedor actual y volver a crearlo con los puertos mapeados con puertos del host
    * Borrar el contenedor mysql-server
    ```
    docker stop mysql-server
    docker rm mysql-server
    docker ps -a # verificar que fue borrado
    ```
    * Volver a crear el contenedor mapeando los puertos
    ```
    docker run --name mysql-server -e MYSQL_ROOT_PASSWORD='1234' -d -p 43306:3306 mysql
    ```

6. Comprobar los puertos en host, contenedor y mapeo

* Puertos en el host
```
ss -ln | grep 3306
```

* Puertos en Contenedor y Mapeo
```
docker ps
docker port mysql-server
```

7. Probar conexion mysql remota  
    1. Desde el host (Opcional**requiere instalación de cliente mysql**)
    ```
    mysql -u root -p -h 127.0.0.1 -P 3306
    ```
    2. Desde un contenedor cliente mysql
    ```
    docker run -it --rm mysql mysql -h<hostIP> -uroot -p  -P3306
    ```

    
## Variables de Entorno (ejercicios)

1. Buscar en la documentación de la imagen la descripcion del uso de la variable ```MYSQL_DATABASE```

2. Crear contenedor pasándole la variable de entorno ```MYSQL_DATABASE```
    2.1. Borrar el contenedor mysql-server
    ```
    docker stop mysql-server 
    docker rm mysql-server
    ``` 
    2.2. Volver a arrancar un contenedor mysql-server con la variable de entorno ```MYSQL_DATABASE```
    ```
    docker run --name mysql-server -e MYSQL_ROOT_PASSWORD='1234' -e MYSQL_DATABASE=mibase -d -p 3306:3306 mysql  
    ```

## Persistencia de Datos (ejercicios)
**Utilizando el contenedor del la sección anterior**
1. Agregar nuevos datos en la base de datos  
    1.1. Iniciar sesión mysql de línea de comandos
    ```
    docker run -it --rm mysql mysql -h<hostIP> -uroot -p1234 -P3306 
    ```

    1.2. Cargar datos en la base de datos mibase
    ```
    use mibase;  
    CREATE USER 'usuario1'@'localhost' IDENTIFIED BY '1234';  
    GRANT ALL PRIVILEGES ON mibase.* TO 'usuario1'@'localhost';  
    FLUSH PRIVILEGES;  
    create table tabla1 (id INT AUTO_INCREMENT PRIMARY KEY, nombre VARCHAR(255));  
    insert into tabla1 (nombre) values ('daniel'), ('karina');  
    select * from tabla1;
    ``` 
        
2. Detener el contenedor y volverlo a arrancar y observar qué ocurre con los datos cargados en el punto anterior
```
docker stop mysql-server

docker start mysql-server
```

#### Verificación de datos (aplica para todos los ejercicios que siguen)
Iniciar sesión mysql de línea de comandos
```
docker run -it --rm mysql mysql -h<hostIP> -uroot -p -P3306
```
Verificar los datos cargados en el punto anterior
```
select user from mysql.user;

select * from tabla1;
```


3. Borrar el contenedor, luego arrancar otro idéntico y observar qué ocurrió con los datos cargados en el punto anterior

```
docker stop mysql-server

docker rm mysql-server

docker run --name mysql-server -e MYSQL_ROOT_PASSWORD='1234' -e MYSQL_DATABASE=mibase -d -p 3306:3306 mysql

```
**Que ocurrió con los datos?**


## Volúmenes Docker  (ejercicios)

1. Borrar el contenedor y volver a arrancar uno idéntico pero con un “named” volume asociado a la ruta del contenedor /var/lib/mysql donde se almacena la base de datos.  
    1.1. Borrar el contenedor en ejecución
    ```  
    docker stop mysql-server  
    docker rm mysql-server
    ``` 
    1.2. Arrancar nuevo contenedor indicando un volumen
    ``` 
    docker run --name mysql-server -e MYSQL_ROOT_PASSWORD='1234' -e MYSQL_DATABASE=mibase -v voldatos:/var/lib/mysql -d -p 3306:3306 mysql
    ```
    1.3. Iniciar sesión mysql de línea de comandos
    ```  
    docker run -it --rm mysql mysql -h<hostIP>) -uroot -p -P3306
    ```
    1.4. Modificar datos en mibase
    ```
    use mibase;  
    CREATE USER 'usuario1'@'localhost' IDENTIFIED BY '1234';  
    GRANT ALL PRIVILEGES ON mibase.* TO 'usuario1'@'localhost';  
    FLUSH PRIVILEGES;  
    create table tabla1 (id INT AUTO\_INCREMENT PRIMARY KEY, nombre VARCHAR(255));  
    insert into tabla1 (nombre) values ('daniel'), ('karina');  
    select * from tabla1;  
    quit;
    ```

2. Borrar el contenedor y volver a arrancar uno idéntico y observar qué ocurrió con los datos cargados en el punto anterior
```
docker stop mysql-server
docker rm mysql-server

docker run \--name mysql-server \-e MYSQL\_ROOT\_PASSWORD='1234' \-e MYSQL\_DATABASE=mibase \-v voldatos:/var/lib/mysql \-d \-p 3306:3306 mysql
```
**Que ocurrió con los datos?**


### Bind Mounts
    
1. Borrar el contenedor y volver a arrancar uno idéntico pero con un “bind mount” asociado a la ruta del contenedor ```/etc/mysql/conf.d``` donde se almacena configuración personalizada (customized) de mysql  
    1.1. Detener y borrar el contenedor
    ```  
    docker stop mysql-server  
    docker rm mysql-server
    ```
    1.2. Crear directorio para configuración y archivo de configuración customizada
    ```
    mkdir misdatos  
    cd misdatos  
    nano config-file.cnf 
    ``` 
    Agregar las siguientes líneas:
    ```  
    [mysqld]  
    port=3307
    ```
    Arrancar el contenedor agregando el mapeo al puerto nuevo  
    ```
    docker run --name mysql-server -e MYSQL_ROOT_PASSWORD='1234' -e MYSQL_DATABASE=mibase -v voldatos:/var/lib/mysql -v /home/daniel/miconfig:/etc/mysql/conf.d -d -p 3307:3307 -p 3306:3306 mysql  
    ```
    1.3. Verificar los puertos expuestos y mapeados
    ``` 
    ss -ln | grep 330  
    docker port mysql-server  
    docker ps
    ```
    1.4. Iniciar conexión remota al nuevo puerto
    ``` 
    docker run -it --rm mysql mysql -h<hostIP> -uroot -p -P3307  
    ```
    1.5. Editar el archivo config-file.cnf configurando port=3306
    ```  
    nano config-file.cnf
    ```
    Editar:
    ```
    [mysqld]  
    port=3306
    ```

    1.6. Reiniciar el contenedor y verificar que funciona el puerto correcto
    ```  
    docker stop mysql-server  
    docker start mysql-server  
    docker run -it --rm mysql mysql -h<hostIP> -uroot -p -P3306
    ```
2. Inicialización de una instancia nueva mysql colocando archivos y scripts sql en el directorio /docker-entrypoint-initdb.d del contenedor.

    2.1. Como paso preliminar, creamos un sqldump.
    
    * Verificar que la base de datos tenga información antes
    ```    
    docker run -it --rm mysql mysql -h<hostIP> -uroot -p -P3306 

    # en consola mysql
    use mibase;  
    show tables;  
    quit;
    ```
    
    * Realizar un ```mysqldump``` y volcarlo a un archivo en el directorio actual en el host
    ```  
    docker exec -it mysql-server mysqldump -u root -p1234 mibase > mibasedump.sql
    ```

    2.2. Solución con un bind mounts. Discutirla

    

## Dockerfiles (Referencia resumida)

Un Dockerfile es un documento de texto que contiene todos los comandos que un usuario podría llamar en la línea de comandos para ensamblar una imagen.
La descripción de las instrucciones del Dockerfile se pueden ver en la siguiente referencia: 

[https://docs.docker.com/engine/reference/builder/](https://docs.docker.com/engine/reference/builder/)


### Formato del Dockerfile
```
# Commentarios

FROM <image_base>:<tag>

INSTRUCCIÓN-1 argumentos

INSTRUCCIÓN-2 argumentos

...

INSTRUCCIÓN-n argumentos
```

Docker ejecuta las instrucciones del Dockerfile en orden.

**El Dockerfile debe empezar siempre con la instrucción ```FROM```**


### Algunas Instrucciones:

##### FROM 
Especifica la imagen base desde la que se construye la nueva imagen.  

```FROM <imagen>```

##### Comentarios
Docker trata a las líneas que comienzan con \# como comentarios.  
```# Commentario```

##### RUN

Ejecuta comandos y aplica los resultados en la imagen.

```RUN <comando>```

##### COPY
Copia archivos desde directorios del host al filesystem de la imagen.  

```COPY <src> <dest>```

##### EXPOSE
Informa a Docker que el contenedor escucha en los puertos de red especificados en tiempo de ejecución. 

La instrucción EXPOSE en realidad **no publica el puerto**. Funciona como una especie de documentación entre quien construye la imagen y quien maneja el contenedor, sobre qué puertos se pretende publicar. 

Para publicar realmente el puerto cuando se ejecuta el contenedor, use el indicador -p en la ejecución de docker run. 

```EXPOSE <port>```

##### CMD
Proporciona valores predeterminados para un contenedor en ejecución. Estos valores pueden incluir un ejecutable.

Solo puede haber una instrucción CMD en un Dockerfile. Si se incluye más de un CMD, sólo tendrá efecto el último CMD.

```CMD ["ejecutable","param1","param2"]``` 

### Ejemplo: Solucion  al ejercicio 2 de la sección Bind Mounts

**Se requiere un sqldump llamado ```mibasedump.sql``` tal como el creado en el ejercicio anterior**

1. Editar el archivo Dockerfile (sin extensión)
``` 
nano Dockerfile
```
Agregar:
```
FROM mysql:latest  
    
ENV MYSQL\_ROOT\_PASSWORD=1234  
ENV MYSQL\_DATABASE=mibase  
ENV MYSQL\_USER=usuario  
ENV MYSQL\_PASSWORD=1234  
    
COPY mibasedump.sql /docker-entrypoint-initdb.d/  
EXPOSE 3306
```  
2. Crear una imagen nueva mysql-mibase
``` 
docker build -t mysql-mibase .  
Verificar que se creo una imagen nueva  
docker images
```
3. Crear un contenedor con la imagen nueva
``` 
docker run --name mysql-server -v  misdatos:/var/lib/mysql --rm -d -p 3306:3306 mibase-mysql
```
4. Conectarse a la base de datos y verificar que la información del mysqldump está cargada.


## Ejercicios

1. Crear imagen docker de un servidor web utilizando como base la imagen oficial de Apache y que aloje la página web estática del siguiente link:

    ```wget https://github.com/Unahur-Programacion-Concurrente/StaticWebsite/raw/master/staticweb.tar```  
    
    **Nota**: Tambien lo puede bajar clonando el repositorio github: [https://github.com/Unahur-Programacion-Concurrente/StaticWebsite](https://github.com/Unahur-Programacion-Concurrente/StaticWebsite)  
    
    
    1.1. Crear el proyecto y clonar el código  
    ```
    mkdir -p staticweb/src  
    cd staticweb/  
    wget https://github.com/Unahur-Programacion-Concurrente/StaticWebsite/raw/master/staticweb.tar  
    tar xvf staticweb.tar -C src
    ```

    1.2. Utilizar la imagen oficial de Apache como base para crear una imagen personalizada que contenga el código de la página en su filesystem.

    ***Se detallan a continuación los pasos a seguir, pero no los detalles, que deben buscarlos en las referencias.***

    1.2.1. Buscar la imagen oficial de Apache en docker hub.
    
    1.2.2. Utilizando como base la versión oficial de Apache (última), crear un Dockerfile de modo que:
 
    - Tenga como imagen base la imagen oficial de Apache. 

    - Copie todo el contenido del directorio ```~/staticweb/src``` en el directorio ```/usr/local/apache2/htdocs/``` de la imagen.

    1.2.3 Crear (docker build) una imagen llamada staticweb utilizando el Dockerfile.

    1.2.4. Verificar que la imagen fué creada correctamente (docker images)

2. Crear un contenedor utilizando la imagen creada en el ejercicio 1.

   ***Se detallan a continuación los pasos a seguir, pero no los detalles, que deben buscarlos en las referencias.***

    2.1. Crear un contenedor (run) de modo que:

    * Que tenga nombre staticweb-prod

    * Mapee el puerto 8081 del host al puerto 80 del contenedor.

    * Que el contenedor se borre automáticamente cuando se detenga.

    2.2. Verificar que el contenedor esté creado correctamente y funcionando.

    2.3. Probar que la aplicación está funcionando, abriendo una sesión http a la dirección del host y puerto 8080.

    2.4. **Discutir en clase:**

    - Si hubiera que actualizar el código de la pagina, que opciones hay para hacerlo?.

3. Crear un contenedor de modo que el contenido de la página es accesible desde host para que el desarrollador pueda editarlo y los cambios realizados se reflejan inmediatamente.

   ***Se detallan a continuación los pasos a seguir, pero no los detalles, que deben buscarlos en las referencias.***

    3.1. Crear un contenedor (docker run) de modo que:

    * Que tenga nombre staticweb-dev

    * monte (mount bind) el directorio ```~/staticweb/src``` en el directorio ```/usr/local/apache2/htdocs/``` del contenedor.

    * Mapee el puerto 8080 del host al puerto 80 del contenedor.

    * Que el contenedor se borre automáticamente cuando se detenga.

   3.2. Verificar que el contenedor esté creado correctamente y funcionando.

   3.3. Probar que la aplicación está funcionando, abriendo una sesión http a la dirección del host y puerto 8080.

   3.4. Realizar modificaciones en los archivos fuente y verificar que se ven los cambios al recargar la página en el browser.
   
   3.5. **Discutir en clase:**

    - Si tuviera que crear un contenedor con la nueva versión de página en otro host, que opciones tendria?
    - Que ocurre con las páginas web en el directorio ```/usr/local/apache2/htdocs/``` de la **imagen** ?


4. Construir una imagen que incluya la actualización del código del sitio web.

    4.1.1 Crear una nueva imagen que contenga los cambios realizados en el ejercicio 3 y asignarle el nombre staticweb:v2

5. Crear un nuevo contenedor con la nueva versión de l aimagen.

    5.1. Detener y borrar el contenedor staticweb-prod y 
    5.2. Volver a crearlo con la imagen nueva staticweb:v2, de modo que el código de la página **solo sea accesible desde el contenedor y no se pueda modificar desde el host.**



# Aplicaciones con Múltiples Contenedores

Es posible (y lo más común) desplegar aplicaciones que involucren varios contenedores.

Los contenedores pueden compartir volúmenes de almacenamiento y se pueden comunicar por medio de redes de docker.

## Ejercicios

**Borrar todos los contenedores e imagenes de ejercicios anteriores**

#### Implementar un sitio Wordpress utilizando los siguientes contenedores a partir de sus imágenes oficiales:

- MySQL (versión 8.0)  
- Wordpress (versión 5.1.1-fpm-alpine)  
- Nginx (versión 1.15.12-alpine)

Crear los siguientes named volumes para persistencia de datos:

- dbdata (para mysql)  
- wordpress (para los archivos de wordpress)

Los contenedores se comunicarán por el siguiente network:

- app-network


### Procedimiento

1. Crear un directorio para el proyecto
    ```
    mkdir ~/wordpress
    cd ~/wordpress
    ```

2. Crear archivo de variables de entorno  
   ```
   nano .env  
   ```

   Agregar las siguientes líneas:  
   ```
   MYSQL\_ROOT\_PASSWORD=password  
   MYSQL\_USER=wpuser  
   MYSQL\_PASSWORD=password
   ```
3. Crear la red (docker network) app-network

4. Crear el contenedor MySql de modo que:  
   * utilice la imagen base mysql:8.0  
   * que tenga el nombre db  
   * que se le pase al contenedor las variables de entorno del archivo .env y la variable ```MYSQL_DATABASE=wordpress```  
   * que monte en un named volume (dbdata) el directorio /var/lib/mysql del contenedor.  
   * que se conecte a la red app-network  
   * que arranque en modo detach  
   * que se borre automáticamente si se detiene  
   * que envíe al script de arranque el comando: ```--default-authentication-plugin=mysql_native_password```

   ```
   docker run --name db --env-file .env -e MYSQL_DATABASE=wordpress --network app-network --rm -v dbdata:/var/lib/mysql -d mysql:8.0 --default-authentication-plugin=mysql_native_password
   ```

5. Verificar que el contenedor esté arriba realizando una conexión de un cliente por la red app-network.

6. Crear el contenedor Wordpress de modo que:  
   * utilice la imagen base ```wordpress:5.1.1-fpm-alpine```  
   * que tenga el nombre wordpress  
   * que se le pase al contenedor las variables de entorno del archivo .env y las variables:
   ```
   WORDPRESS_DB_HOST=db:3306
   WORDPRESS_DB_USER=wpuser
   WORDPRESS_DB_PASSWORD=password
   WORDPRESS_DB_NAME=wordpress
   ```
   * que monte en un named volume (wordpress) el directorio /var/www/html  del contenedor.  
   * que se conecte a la red app-network  
   * que arranque en modo detach  
   * que se borre automáticamente si se detiene

   ```
   docker run --name wordpress --env-file .env \  
   -e WORDPRESS_DB_HOST=db:3306 \  
   -e WORDPRESS_DB_USER=wpuser \  
   -e WORDPRESS_DB_PASSWORD=1234 \  
   -e WORDPRESS_DB_NAME=wordpress \  
   -v wordpress:/var/www/html --network app-network -d --rm wordpress:5.1.1-fpm-alpine
   ```

7. Observar los logs (```docker logs```) del contenedor y constatar que se pudo conectar a la base de datos en forma exitosa.  
     
8. Crear un directorio para la configuración personalizada de nginx.
   ```
   mkdir ~/nginx-conf
   ```  
9. Crear el archivo de configuración personalizada de nginx  
**Nota: tambien puede bajar el archivo del repositorio git.**
   ```
   nano ~/nginx-conf/nginx.conf  
   ```
   Agregar las siguientes líneas:  
```
server {
        listen 80;
        listen [::]:80;

        server_name www.ejemplo.com;

        index index.php index.html index.htm;

        root /var/www/html;

        location / {
                try_files $uri $uri/ /index.php$is_args$args;
        }

        location ~ \.php$ {
                try_files $uri =404;
                fastcgi_split_path_info ^(.+\.php)(/.+)$;
                fastcgi_pass wordpress:9000;
                fastcgi_index index.php;
                include fastcgi_params;
                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                fastcgi_param PATH_INFO $fastcgi_path_info;
        }

        location ~ /\.ht {
                deny all;
        }

        location = /favicon.ico {
                log_not_found off; access_log off;
        }
        location = /robots.txt {
                log_not_found off; access_log off; allow all;
        }
        location ~* \.(css|gif|ico|jpeg|jpg|js|png)$ {
                expires max;
                log_not_found off;
        }
}
```
10. Crear el contenedor Nginx de modo que:  
    * utilice la imagen base ```nginx:1.15.12-alpine```  
    * que tenga el nombre ```webserver```  
    * que monte el named volume (```wordpress```) que utiliza el contenedor wordpress el directorio ```/var/www/html```  del contenedor.  
    * que haga un mount bind del directorio ```~/nginx-conf``` del host en el directorio ```/etc/nginx/conf.d``` del contenedor.  
    * que se conecte a la red ```app-network```  
    * que arranque en modo detach  
    * que mapee el puerto 80 del host con el puerto 80 del contenedor  
    * que se borre automáticamente si se detiene
```
docker run --name webserver -p 80:80 -v wordpress:/var/www/html -v /home/daniel/wordpress/nginx-conf:/etc/nginx/conf.d --network app-network --rm -d nginx:1.15.12-alpine
```

# Docker Compose

Docker Compose es una herramienta para definir y ejecutar aplicaciones de Docker de varios contenedores. 

En Compose, se usa un archivo YAML para configurar los servicios de la aplicación. 

Después, con un solo comando, se crean y se inician todos los servicios de la configuración.

## Instalación

sudo curl \-L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname \-s)-$(uname \-m)" \-o /usr/local/bin/docker-compose

sudo chmod \+x /usr/local/bin/docker-compose

docker-compose \--version

## docker-compose.yaml
```
version: '3'
services:
  db:
    image: mysql:8.0
    container_name: db
    restart: unless-stopped
    env_file: .env
    environment:
      - MYSQL_DATABASE=wordpress
    volumes:
      - dbdata:/var/lib/mysql
    command: '--default-authentication-plugin=mysql_native_password'
    networks:
      - app-network

  wordpress:
    depends_on:
      - db
    image: wordpress:5.1.1-fpm-alpine
    container_name: wordpress
    restart: unless-stopped
    env_file: .env
    environment:
      - WORDPRESS_DB_HOST=db:3306
      - WORDPRESS_DB_USER=$MYSQL_USER
      - WORDPRESS_DB_PASSWORD=$MYSQL_PASSWORD
      - WORDPRESS_DB_NAME=wordpress
    volumes:
      - wordpress:/var/www/html
    networks:
      - app-network

  webserver:
    depends_on:
      - wordpress
    image: nginx:1.15.12-alpine
    container_name: webserver
    restart: unless-stopped
    ports:
      - "80:80"
    volumes:
      - wordpress:/var/www/html
      - ./nginx-conf:/etc/nginx/conf.d
    networks:
      - app-network
volumes:
	dbdata:
	wordpress:
networks:
	app-network:
		driver: bridge
```