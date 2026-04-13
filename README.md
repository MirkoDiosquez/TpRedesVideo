# comoEsRecamAhhhh

A — Servidor en puerto distinto
Crear un sitio web simple (HTML)
Hacer que funcione en un puerto raro, por ejemplo:
http://localhost:8080

👉 Básicamente: que el servidor no use el puerto 80 clásico.



-Primer paso : Abrimos la "máquina virtual" con el  
 docker run -ri -p 8080:80 ubuntu:latest bash

 Segundo paso : Instalar Nginx.
 apt update
 apt install nginx

 Tercer paso : Corroborar que este corriendo el nginx.
 
 service nginx status
 
 si no corre, correrlo:

 service nginx start.

 Una vez el servidor corre, Tenemos que acceder a la ip para poder ver el servidor web corriendo. Accedemos con el dominio de:

 http://localhost:8080.


 Veremos la página predeterminada de Nginx. Para modificarla, accedemos al archivo HTML con nano:

 nano /var/www/html/index.nginx-debian.html

 Ahí podemos editar el contenido de la página a gusto. Una vez realizados los cambios, guardamos con Ctrl+O y salimos con Ctrl+X.



 B — Configuración por path
Servir el sitio estático B en un subdirectorio del servidor principal. El sitio debe ser accesible desde: http://localhost/static


Primer paso : Abrimos la "Máquina virutal" con el docker.
docker run -ri -p 8585:85 ubuntu:latest bash

( No es lo más recomendable cambiar el puerto interno )

Esto te deja dentro del contenedor listo para instalar cosas.

Repetimos los mismos paso.

apt update
apt install nginx
apt install nano

service nginx status
service nginx start.


Ahi ya tenemos todo

Ahora hacemos algo distinto.

Segundo paso: Creamos una carpeta en cualquier parte. En nuestro caso, la creamos en /var

mkdir carpetaAuxiliar <- nombre de la carpeta que vamos a utilizar.

Tercer paso : hacemos el archivo html y ponemos cosas adentro para verlo proximamente en la página.

nano /var/carpetaAuxiliar/index.html

Cuarto paso: Una vez hecho el nano, escribimos 3 cosas para ver que nos va a salir y guardamos. Despues de eso escribimos el siguiente comando.

nano /etc/nginx/sites-available/default


ese comando sirve para cambiar la ruta del archivo donde abre la carpeta el cual salta la página predeterminada de Nginx.


nos devolvera algo asi


server {
    listen 80;
listen [::]:80;

location {
    alias /var/www/html;
}

}


Nosotros lo modificamos asi

listen 85;
listen [::]:85;

location /static/ {
    alias /var/carpetaAuxiliar/;
}


Pasos Opcionales :

ls -ld /var/carpetaAuxiliar/
nginx -t

Ambos 2 son para ver si tiene permisos para ver los permisos actuales y si existe un error.


Cuarto paso : Reiniciar nginx 

service nginx restart 

Quinto paso : Corroborar si funciona

http://localhost:8585/static/


C — Configuración por dominio (Virtual Host)
Configurar un Virtual Host para que el sitio C sea accesible mediante un nombre de dominio local. Para esto:

// ESTE PASO LO HACEMOS DESDE VIRTUAL BOX

Paso 1 — Instalar NGINX
bashsudo apt update
sudo apt upgrade -y
sudo apt install nginx -y
Verificar que esté corriendo:
bashsudo systemctl status nginx

Paso 2 — Crear la carpeta y el sitio estático
bashsudo mkdir -p /var/www/misitio
sudo nano /var/www/misitio/index.html
Contenido del HTML:
html<html>
  <body>
    <h1>Bienvenido a misitio.com</h1>
  </body>
</html>

Paso 3 — Crear el Virtual Host en NGINX
bashsudo nano /etc/nginx/sites-available/misitio
Contenido del archivo:
nginxserver {
    listen 80;
    server_name misitio.com;

    root /var/www/misitio;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}

Paso 4 — Activar el sitio y deshabilitar el default
bashsudo rm /etc/nginx/sites-enabled/default
sudo ln -s /etc/nginx/sites-available/misitio /etc/nginx/sites-enabled/
Verificar que la configuración no tenga errores:
bashsudo nginx -t
Recargar NGINX:
bashsudo systemctl reload nginx

Paso 5 — Configurar /etc/hosts
bashsudo nano /etc/hosts
Agregar esta línea:
127.0.0.1 misitio.com

Paso 6 — Verificar que funciona
bashcurl http://misitio.com
Resultado esperado: el HTML con "Bienvenido a misitio.com".



D — Crear una aplicación Java Spring Boot mínima (Hola Mundo) y correrla en el servidor en un puerto distinto al 80 (ej: 8000)
Configurar el servidor web (Apache o NGINX) para actuar como reverse proxy hacia la aplicación Java que corre en localhost:8000.
La aplicación debe ser accesible desde: http://<IP-del-servidor>/app/

El servidor web debe redirigir internamente las peticiones a /app/ hacia la aplicación Java que está corriendo en el puerto 8000.


Tutorial: Spring Boot + NGINX Reverse Proxy con Docker

Contexto inicial
Este tutorial asume que tenés Docker instalado. Todo se hace desde una terminal. No necesitás instalar Java ni Maven en tu máquina real, todo corre dentro del contenedor.

Paso 1 — Levantar un contenedor Ubuntu
bashdocker run -it --name mi-servidor -p 8888:80 -p 9000:8000 ubuntu:latest bash
Esto levanta un Ubuntu limpio y te mete adentro. El prompt cambia a algo como root@xxxxxxx:/#.
Qué hace cada parte:

-it → modo interactivo (podés escribir comandos)
--name mi-servidor → le da un nombre al contenedor
-p 8888:80 → mapea el puerto 80 del contenedor al 8888 de tu máquina
-p 9000:8000 → mapea el puerto 8000 del contenedor al 9000 de tu máquina


Paso 2 — Instalar Java, Maven y NGINX
bashapt update && apt install -y openjdk-17-jdk maven nginx nano curl
Cuando pregunte la región, elegí America → Buenos_Aires.
Verificar que se instaló todo:
bashjava -version && mvn -version && nginx -v
Deberías ver las versiones de cada uno sin errores.

Paso 3 — Crear la estructura del proyecto
bashmkdir -p ~/holamundo/src/main/java/com/example
mkdir -p ~/holamundo/src/main/resources
cd ~/holamundo

Paso 4 — Crear los archivos de la app
4.1 — App.java (el controlador que responde "Hola Mundo")
bashnano src/main/java/com/example/App.java
javapackage com.example;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
@RestController
public class App {

    public static void main(String[] args) {
        SpringApplication.run(App.class, args);
    }

    @GetMapping("/")
    public String holaMundo() {
        return "Hola Mundo desde Spring Boot!";
    }
}
Guardar: Ctrl+O → Enter → Ctrl+X

4.2 — application.properties (configura el puerto y la ruta)
bashnano src/main/resources/application.properties
server.port=8000
server.servlet.context-path=/app
Guardar: Ctrl+O → Enter → Ctrl+X

4.3 — pom.xml (dependencias del proyecto Java)
bashnano pom.xml
xml<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>holamundo</artifactId>
    <version>1.0</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
Guardar: Ctrl+O → Enter → Ctrl+X

Paso 5 — Compilar el proyecto
bashmvn clean package -DskipTests
Tarda unos minutos porque descarga las dependencias. Al final tiene que aparecer BUILD SUCCESS.

Paso 6 — Arrancar la app Spring Boot
bashjava -jar target/holamundo-1.0.jar &
El & la corre en segundo plano para poder seguir usando la terminal. Esperás a ver Started App in X seconds y verificás que funciona:
bashcurl http://localhost:8000/app/
Deberías ver: Hola Mundo desde Spring Boot!

Paso 7 — Configurar NGINX como Reverse Proxy
bashnano /etc/nginx/sites-available/default
Borrás todo y pegás esto:
nginxserver {
    listen 80;

    location /app/ {
        proxy_pass http://localhost:8000/app/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
Guardar: Ctrl+O → Enter → Ctrl+X
Iniciás NGINX:
bashservice nginx start

Paso 8 — Verificar que todo funciona
bashcurl http://localhost/app/
Deberías ver de nuevo: Hola Mundo desde Spring Boot!
La diferencia con el paso 6 es que ahora la petición entra por el puerto 80 de NGINX, que la redirige internamente al puerto 8000 de Spring Boot. El usuario final solo ve /app/, nunca el puerto 8000.