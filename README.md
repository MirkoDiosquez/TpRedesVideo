# comoEsRecamAhhhh

A — Servidor en puerto distinto
Crear un sitio web simple (HTML)
Hacer que funcione en un puerto raro, por ejemplo:
http://localhost:8080

👉 Básicamente: que el servidor no use el puerto 80 clásico.



-Primer paso : Abrimos la "máquina virtual" con el  
 docker run -ri -p 8080:80

 Segundo paso : Instalar Nginx.
 apt update
 apt install nginx

 Tercer paso : Corroborar que este corriendo el nginx.
 
 service nginx status
 
 si no corre, correrlo:

 service nginx start.

 Una vez el servidor corre, Tenemos que acceder a la ip para poder ver el servidor web corriendo. Accedemos con el dominio de:

 http://localhost:8080.


 Veremos la página con el html básico. Modificar con algunas cosas a gusto.
