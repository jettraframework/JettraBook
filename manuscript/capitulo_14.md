# Capítulo 14

Este capítulo trata sobre contenedores Docker. Aprenderás a crear y configurar imágenes Docker para sus proyectos Java, y cómo subirlas a Docker Hub.

Este capítulo incluye los temas:

* Introducción a Docker

* Creación y configuración de DockerFile

* Solucionando fallas

* Establecer Conexión a MongoDB a través de Docker

* Gestión de DockerHub

## Introducción a Docker

[Docker](https://www.docker.com/), según su sitio oficial, es una plataforma diseñada para ayudar a los desarrolladores a crear, compartir y ejecutar aplicaciones en contenedores.

Para este capítulo, continuaremos utilizando el proyecto que desarrollamos en el capítulo 11.



### Creación y configuración de DockerFile

Un archivo DockerFile, es un archivo de texto plano en que él se indican las instrucciones necesarias para crear una imagen de Docker. La documentación oficial se puede encontrar en [https://docs.docker.com/engine/reference/builder/](https://docs.docker.com/engine/reference/builder/)


Para crear un archivo DockerFile desde Apache NetBeans, siga estos pasos:

1. Ubíquese en la pestaña File en el ménu superior.

2. Haga clic derecho en el proyecto

3. Seleccione New File en el menú desplegable

4. En la ventana emergente, selecciona Other en Categories y DockerFile en File Types

![](figura_14_00.png)

5. Haga clic en el botón Next y no cambie el nombre de archivo

![](figura_14_01.png)

6. Haga clic el botón **Finish** para generar el archivo con el siguiente contenido

```yaml

FROM alpine:latest

CMD ["/bin/sh"]


```




Antes de crear una imagen de Payara Micro, asegúrese de revisar las imágenes disponibles en Docker Hub.

 Ingrese a [https://hub.docker.com/r/payara/micro](https://hub.docker.com/r/payara/micro) para obtener la última versión disponible, utilice el comando **docker pull payara/micro:latest**, tal como se muestra en la siguiente figura


![](figura_14_02.png)

Al momento de escribir este libro, la versión más reciente es la siguiente:

```

docker pull payara/micro:6.2024.11-jdk21

```

En el archivo docker-file, reemplace el contenido con el correspondiente a la imagen de Payara Micro, tal como se muestra a continuación:

```yaml

FROM payara/micro:6.2024.11-jdk21

COPY target/capitulo11.war $DEPLOY_DIR

# CMD ["--nocluster","--deploymentDir", "/opt/payara/deployments", "--contextroot", "capitulo11"]

```


Ejecute el siguiente comando:


```shell

mvn clean verify

```


Posteriormente, construya la imagen utilizando el comando:

```shell

docker build -t avbravo/capitulo11 .

```

Espere un momento mientras se descarga de la imagen de payara/micro:6.2024.11-jdk21, como se muestra en la figura siguiente:

![](figura_14_03.png)


### Ver imágenes Docker

Para visualizar las imágenes de Docker disponibles, ejecute el siguiente comando:

```shell

docker images

```

En la siguiente figura se observa un listado de las imágenes de Docker que tenemos instaladas:

![](figura_14_04.png)

Identifique la imagen denominada avbravo/capitulo11. y copie el IMAGE ID correspondiente, en este caso es **d1d2aaf7f687**, como se muestra a continuación:

```shell

REPOSITORY           TAG              IMAGE ID       CREATED              SIZE
avbravo/capitulo11   latest           d1d2aaf7f687   About a minute ago   394MB


```

Para ejecutar la imagen en el puerto 8080, introduzca el siguiente comando en su terminal:

```shell

docker run -d --rm -p 8080:8080 --name capitulo11 avbravo/capitulo11

```

Abra su navegador y acceda a la URL [http://localhost:8080/capitulo11/](http://localhost:8080/capitulo11/)


Debería ver el mensaje **Hello from Payara!** en su navegado, lo que indica que está funcionando nuestra imagen de Payara Micro

![](figura_14_05.png)



Para detener la ejecución, siga los siguientes pasos:

1. Determine el ID de la imagen  (CONTAINER ID) mediante el siguiente comando

```shell

docker ps -a

```

Esto muestra el listado de todas las imágenes. Localice el **CONTAINER ID** de la imagen avbravo/capitulo11. En este ejemplo, el CONTAINER ID es **0b069d72b69a**


```
                                                       
CONTAINER ID   IMAGE                COMMAND                  CREATED         STATUS                  PORTS                                                 NAMES                           
0b069d72b69a   avbravo/capitulo11   "/bin/sh entrypoint.…"   2 minutes ago   Up 2 minutes            6900/tcp, 0.0.0.0:8080->8080/tcp, :::8080->8080/tcp   capitulo11                      
be67313bb936   mongo:4.4            "docker-entrypoint.s…"   3 days ago      Up About an hour        0.0.0.0:27017->27017/tcp, :::27017->27017/tcp         v44_mongodb_container_1         
c63227ece9e4   mongo:latest         "docker-entrypoint.s…"   4 days ago      Exited (0) 3 days ago                                                         lasted_mongodb_container_1  

```

2. Para detener la ejecución, utilice el siguiente comando:

```shell

docker stop 0b069d72b69a

```
o

```shell

docker stop capitulo11

```

Si ahora ejecuta el comando **docker ps -a**, notará que la imagen ya no aparece en la lista de imágenes. Recuerde que es posible volver a ejecutar la imagen mediante **docker run**, tal como se realizo anteriormente.

Para volver a ejecutar la imagen, utilice el siguiente comando:

```shell

docker run -d --rm -p 8080:8080 --name capitulo11 avbravo/capitulo11

```


Para realizar una consulta al endpoint **/api/estudiante/**, asegúrese de incluir **/capitulo11**  en la URL. Por ejemplo:

```shell

curl --location --request GET http://localhost:8080/capitulo11/api/estudiante/1-2-3


```
Si la respuesta es vacía o se produce un error, significa que la aplicación no está devolviendo los documentos esperados. En este caso, es necesario identificar el problema que impide obtener los resultados deseados.


```shell

avbravo@avbravo-IdeaPad-Gaming-3-15ARH7:~$ curl --location --request GET http://localhost:8080/capitulo11/api/estudiante/1-2-3

<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
  <title>Payara Micro #badassfish - Error report</title>
  <style type="text/css">
   <!--H1 {font-family:Tahoma,Arial,sans-serif;color:white;background-color:#525D76;font-size:22px;}
       H2 {font-family:Tahoma,Arial,sans-serif;color:white;background-color:#525D76;font-size:16px;} 
       H3 {font-family:Tahoma,Arial,sans-serif;color:white;background-color:#525D76;font-size:14px;} 
       BODY {font-family:Tahoma,Arial,sans-serif;color:black;background-color:white;}
       B {font-family:Tahoma,Arial,sans-serif;color:white;background-color:#525D76;}
       P {font-family:Tahoma,Arial,sans-serif;background:white;color:black;font-size:12px;}
       A {color : black;}HR {color : #525D76;}
   -->
</style>
 </head>
<body>
  <h1>HTTP Status 404 - Not Found</h1>
  <hr/>
  <p>
    <b>type</b> Status report
   </p>
   <p><b>message</b>Not Found</p>
   <p><b>description</b>The requested resource is not available.</p><hr/>
   <h3>Payara Micro #badassfish</h3>
</body>
</html>


```



## Solucionando fallas

Para identificar posibles causas de error, es necesario revisar el archivo de registro (log) de la imagen. 

![](figura_14_06.png)

Ejecute el siguiente comando para visualizar este archivo



```shell

 docker logs -f capitulo11

```



Tras analizar el archivo de registro, identificamos el error que impide que se muestren los documentos de los estudiantes. Este problema surge debido a que no es posible establecer conexión con la base de datos MongoDB.

```shell

[ EstudianteRepositoryImpl.findByPk Timed out after 30000 ms while waiting to connect. Client view of cluster state is {type=UNKNOWN, servers=[{address=localhost:27017, type=UNKNOWN, state=CONNECTING, exception={com.mongodb.MongoSocketOpenException: Exception opening socket}, caused by {java.net.ConnectException: Connection refused}}]]

```


Para visualizar el archivo de registro basado en tiempos específicos, utilice la opción **--until=**, como se muestra en el ejemplo siguiente:

```shell

 docker logs -f --until=50s capitulo11

```

## Establecer Conexión a MongoDB a través de Docker

El error con la conexión a MongoDB se produce debido a que se está ejecutando MongoDB en otra imagen de Docker. Por lo tanto, necesitamos configurar la imagen avbravo/capitulo11 para que utilice la conexión a MongoDB.


Como recordará, en el archivo de configuración microprofile-config.properties, establecimos la conexión a la base de datos utilizando la propiedad mongodb.uri. Por lo tanto, pasaremos el URI de conexión a la base de datos durante la ejecución de la imagen.


Detenga la imagen que se está ejecutando,  asegúrese de identificar el CONTAINNER ID:


```shell

docker ps -a

```

Localice el CONTAINER ID

```shell

CONTAINER ID   IMAGE                COMMAND                  CREATED          STATUS                  PORTS                                                 NAMES
ba13a3852bc2   avbravo/capitulo11   "/bin/sh entrypoint.…"   33 minutes ago   Up 33 minutes           6900/tcp, 0.0.0.0:8080->8080/tcp, :::8080->8080/tcp   capitulo11

```


Proceda a detener la imagen utilizando el siguiente comando:

```shell

docker stop ba13a3852bc2

```

Si esta utilizando Ubuntu, puede verificar la dirección IP de su computador utilizando  el comando **hostname -I**. La primera dirección IP que aparece en el resultado corresponde a la dirección IP de su computadora:

```shell

hostname -I

```

Ejemplo del resultado obtenido

```shell

192.168.1.179  

```

Inicie la imagen Docker, agregando el parámetro -e con el valor de mongodb.uri correspondiente al IP encontrado:


```shell

docker run -d --rm -p 8080:8080 -e mongodb.uri=mongodb://192.168.1.179:27017 --name capitulo11 avbravo/capitulo11


```

Si utiliza credenciales para la base de datos debe espeficicarlas en mongodb.uri

```java
docker run -d --rm -p 8080:8080 -e mongodb.uri=mongodb://user:pass@192.168.1.179:27017 --name capitulo11 avbravo/capitulo11

```





Realice nuevamente la solicitud al Endpoint utilizando el siguiente comando:

```shell

curl --location --request GET http://localhost:8080/capitulo11/api/estudiante/1-2-3

```
Si la solicitud se realiza correctamente, obtendremos la respuesta esperada, lo que indica que la conexión a MongoDB se ha establecido exitosamente:


```json

{"edad":49,"idestudiante":"1-2-3","nombre":"Aristides"}

```


Si ya ha especificado una conexión a una plataforma como MongoDB Atlas o a un servidor, no es necesario cambiar este valor.

Por favor, detenga la imagen Docker que está en ejecución e identifique el CONTAINER ID.

```shell

docker ps -a


CONTAINER ID   IMAGE                COMMAND                  CREATED          STATUS          
225555f6462b   avbravo/capitulo11   "/bin/sh entrypoint.…"   23 minutes ago   Up 23 minutes  


docker stop 225555f6462b

```




## Gestión de DockerHub

Dockerhub es la mayor biblioteca de imágenes de contenedores Docker del mundo.

En esta sección, aprenderá a cargar una imagen a Dockerhub de manera que esté disponible para su descarga.


Los pasos que realizaremos son los siguientes:


1. Crear una cuenta en Dockerhub.


2. Subir la imagen capitulo11.


3. Descargar la imagen **capitulo11* desde Dockerhub.



Visite el sitio [https://hub.docker.com/](https://hub.docker.com/) para crear una cuenta nueva o iniciar sesión si ya posee una.

A continuación, ejecute el siguiente comando, asegurandose de reemplazar 'username' con su nombre de ususario en Dockerhub:

```shell

docker login -u username

```


Después de presionar enter, se solicita que ingrese su contraseña. 

Para cargar la imagen al repositorio, ejecute el comando **push** especificando el nombre de usuario y el nombre de la imagen. 

En el ejemplo siguiente el nombre  de usuario es avbravo y el nombre de la imagen es capitulo11.

```shell

 docker push avbravo/capitulo11


```

Tal como se muestra en la figura siguiente.

![](figura_14_07.png)


Una vez que el proceso de carga de la imagen a DockerHub se haya completado, acceda a su cuenta para visualizar la imagen en su repositorio.

![](figura_14_08.png)

Haga clic en la imagen etiquetada como avbravo/capitulo11

![](figura_14_09.png)



Para las futuras versiones, utilice un 'tabname' al realizar 'push' para una identificación más clara de las versiones.


```shell

docker push avbravo/capitulo11:tagname

```

Haga clic en la pestaña tab

![](figura_14_10.png)



Para descargar la imagen desde DockerHub, por favor, ejecute el comando que se proporciona a continuación

```shell

docker pull avbravo/capitulo11:latest

```


Inicie la imagen con el siguiente comando

```shell

docker run -d --rm -p 8080:8080 -e mongodb.uri=mongodb://192.168.1.179:27017 --name capitulo11 avbravo/capitulo11

```

Realice nuevamente la solicitud al endpoint

```shell

curl --location --request GET http://localhost:8080/capitulo11/api/estudiante/1-2-3

```

Recibiremos la respuesta esperada

```json
{"edad":49,"idestudiante":"1-2-3","nombre":"Aristides"}Crear conexión a MongoDB a través de Docker

```

Por favor, detenga la imagen que se está ejecutando, asegurándose de identificar correctamente el CONTAINER ID

```shell

docker ps -a


Identifique el CONTAINER ID


docker stop CONTAINER ID

```

## Resumen
En este capítulo se proporcionó una guía sobre crear imágenes de Docker y subirlas a DockerHub. En el siguiente capítulo, abordaremos la especificación Microprofile RestClient.






