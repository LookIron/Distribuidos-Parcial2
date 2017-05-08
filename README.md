### Examen 1

**Universidad ICESI**  

**Curso:** Sistemas Distribuidos  

**Docente:** Daniel Barragán C.  

**Tema:**  Automatización de infraestructura (Docker) 

**Estudiante:** Esteban Camacho B.

**Código:** A00320168

### Objetivos
* Realizar de forma autónoma el aprovisionamiento automático de infraestructura
* Diagnosticar y ejecutar de forma autónoma las acciones necesarias para lograr infraestructuras estables
* IIntegrar servicios ejecutandose en nodos distintos

### Herramientas utilizadas
* Docker
* Imagenes de Nginx y Http

### Descripción
En la realización del balanceador de carga se utilizó un servidor encargado de realizar 
el balanceo el cual fue configurado con un contenedorcon Nginx y tres servidores web
configurados con index para cargar la muestra de funcionamiento.

### Procedimiento

**1) Configuracion del Dockerfile del servidor Nginx:**

```python
#Se usa la imagen de nginx
FROM nginx

#Se elimina el archivo de configuración default y su carpeta

RUN rm /etc/nginx/conf.d/default.conf && rm -r /etc/nginx/conf.d

#Se agrega el archivo de configuracion de nginx

ADD nginx.conf /etc/nginx/nginx.conf

#Se agrega esta linea para que el contenedor no termine su ejecucion.

RUN echo "daemon off;" >> /etc/nginx/nginx.conf

CMD service nginx start
```

**2) Configuracion del archivo de congiguracion nginx.conf para la distribución:**
```python
worker_processes 4;
 
events { worker_connections 1024; }
 
http {
    sendfile on;
 
    upstream app_servers {
        server web1:80;
        server web2:80;
        server web3:80;
    }
 
    server {
        listen 80;
 
        location / {
            proxy_pass         http://app_servers;
            proxy_redirect     off;
            proxy_set_header   Host $host;
            proxy_set_header   X-Real-IP $remote_addr;
            proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header   X-Forwarded-Host $server_name;
        }
    }
}
```
**3) Configuracion de los servidores web, para ello se modifica el Dockerfile de las diferentes servidores web:**

```python
#Se usa la imagen httpd
FROM httpd
#Se agregar el archivo index.html el cual mostrara la informacion de la web pertinente
ADD index.html /usr/local/apache2/htdocs/index.html
```
**4) Configuracion de del archivo index.html:**
Aqui se pone aquello que quiere que muestre la pagina en el navegador para este caso
```python
<h1> Hola soy la web1 <h1>
```

**5) Configuracion del docker-compose para la gestion de los contenedores:**
``` python
version: '2'

services:
  web1:
    build: ./web1
    expose:
      - "5000"

  web2:
    build: ./web2
    expose:
      - "5000"

  web3:
    build: ./web3
    expose:
      - "5000"

  proxy:
    build: ./nginx
    ports:
      - "8080:80"
    links:
      - web1
      - web2
      - web3
 
```
**6) Para la prueba de funcionamiento se ejecuto el siguiente comando:**

```python
docker-compose -p "hello" up
```
![Uploading parcial2.gif…]()



