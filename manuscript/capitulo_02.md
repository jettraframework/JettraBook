# Capítulo 2

Este capítulo es una introducción a Jmoordbcore, guiándole en la instalación de las dependencias requeridas, enseñándole cómo desarrollar microservicios utilizando Payara Micro.

Este capítulo incluye los temas:

* Jmoordbcore

* Requerimientos

* Docker

* MongoDB

* Generar proyecto Jakarta EE

* Configurar JmoordbCore

* Entidad (Entity)

* Repositorio (Repository)

* Jakarta RESTful Web Services


## Jmoordbcore


Jmoordbcore es un marco de trabajo Java, diseñado para facilitar el desarrollo de aplicaciones que interactúan con bases de datos NoSQL. 


### Características

* Compatible con JakartaEE y Eclipse Microprofile.

* Ofrece una sintaxis intuitiva y fácil de usar.

* Proporciona soporte para bases de datos NoSQL, específicamente MongoDB.

* Utiliza el procesamiento de anotaciones de Java.

* Incluye verificación de sintaxis de escritura.

* Maneja documentos referenciados, embebidos y vistas referenciadas.

* Admite campos secuenciales e incrementales.



## Requerimientos

Para crear un proyecto que emplea Jmoordbcore, es esencial cumplir con los siguientes requisitos:

* APIs Jakarta EE: El proyecto requiere las APIs de Jakarta EE. Puedes obtener más información sobre Jakarta EE en [https://jakarta.ee/](https://jakarta.ee/).

* APIs Microprofile: Las APIs de Microprofile también son necesarias para este proyecto. Aquí puedes encontrar más detalles sobre Microprofile [https://microprofile.io/](https://microprofile.io/).

* Versión Java 21 o superior: Se requiere una versión de Java 21 o superior para el desarrollo de este proyecto.

* Maven y Gradle son herramientas esenciales para la construcción de proyectos. En el Capítulo 1, se detalló el proceso de instalación de Maven. Sin embargo, si está utilizando Apache NetBeans, ya tiene Maven incorporado, por lo que no es necesario instalarlo por separado.

* Servidor Jakarta EE o Microprofile (Payara, WildFly, OpenLiberty, TomEE, Helidon, Quarkus, entre otros).

* En este capítulo, haremos uso de Payara Micro, el cual será instalado a través de plugins de Maven que se añadirán al archivo pom.xml

* Herramientas Postman/RestFox y Curl


**Postman** es una plataforma integral para el desarrollo y uso de APIs. Facilita y optimiza cada etapa del ciclo de vida de las APIs, promoviendo una colaboración eficiente para la creación de APIs de alta calidad.

Puede instalar Postman visitando su sitio web oficial [https://www.postman.com/](https://www.postman.com/).


**RestFox**, es un cliente REST/HTTP ligero.


Instalación en Ubuntu mediante snap.

```shell
sudo snap install restfox

```


**Curl**

Curl es una herramienta de línea de comandos que facilita la transferencia de datos a través de diversas formas de URLs.

Para instalar Curl, descárguelo desde su sitio web oficial [https://curl.se/](https://curl.se/) y siga las instrucciones de instalación.


Asuma que existen estos endpoints

| Método  | EndPoint/URI  |  
|---      |---            |
| GET     |  http://localhost:8080/persona           |   
| GET     |  http://localhost:8080/persona/1         | 
| POST    |  http://localhost:8080/persona           | 
| PUT     | http://localhost:8080/persona/1          | 
| DELETE  |  http://localhost:8080/persona/1         | 




* Invocar el método GET de un Endpoint

```shell
curl -X GET -i http://localhost:8080/personas

```

* Invocar el método GET de un Endpoint pasando un parámetro

```shell
curl -X GET -i http://localhost:8080/persona/1

```

* Invocar el método POST

```shell
curl -X POST -i http://localhost:8080/persona -H 'Content-Type: application/json' -d '{"nombre":"Maria Perez", "idpersona": 2}'
```

* Invocar el método PUT para actualizar

```shell
curl -X PUT -i http://localhost:8080/persona/2 -H 'Content-Type: application/json' -d '{"nombre": "Maria Diaz", "idpersona": 2}'

```


* Invocar el método DELETE

```shell
curl -X DELETE -i http://localhost:8080/persona/2 -H "Accept: application/json"
```

## Siege para pruebas de carga

Siege es una herramienta para probar carga de los sitios Web.

Proceso de instalación

```shell

sudo apt-get install siege -y
```

Ejecutamos una prueba para simular 250 solicitudes

```shell

siege http://localhost:8080/persona -c 250 -r 500

```

Genera una salida con la siguiente información

```json

{	"transactions":			      125000,
	"availability":			      100.00,
	"elapsed_time":			       11.59,
	"data_transferred":		        1.43,
	"response_time":		        0.02,
	"transaction_rate":		    10785.16,
	"throughput":			        0.12,
	"concurrency":			      196.90,
	"successful_transactions":	      125000,
	"failed_transactions":		           0,
	"longest_transaction":		        3.10,
	"shortest_transaction":		        0.00
}

```



## Docker

La definición oficial de Docker , que se puede encontrar en su sitio web [https://www.docker.com/](https://www.docker.com/), es la siguiente:

"**Docker** es una plataforma abierta para desarrollar, distribuir y ejecutar aplicaciones. Docker le permite separar sus aplicaciones de su infraestructura para que pueda entregar software rápidamente. Con Docker, puede gestionar su infraestructura de la misma forma que gestiona sus aplicaciones. Al aprovechar las metodologías de Docker para enviar, probar y desplegar código, puede reducir significativamente el tiempo que transcurre entre la escritura del código y su ejecución en producción."

La definición oficial de **Docker Compose**, según su sitio web [https://docs.docker.com/get-started/08_using_compose/#use-docker-compose](https://docs.docker.com/get-started/08_using_compose/#use-docker-compose) , es la siguiente:

"**Docker Compose** es una herramienta que te ayuda a definir y compartir aplicaciones multicontenedor. Con Compose, puede crear un archivo YAML para definir los servicios y, con un solo comando, puede ponerlo todo en marcha o desmontarlo."

En esta sección, se proporcionarán instrucciones para instalar Docker y Docker Compose en un sistema operativo Ubuntu. Si estás utilizando un sistema operativo diferente, por favor consulta la siguiente guía:[https://docs.docker.com/get-docker/](https://docs.docker.com/get-docker/).

Para la instalación de Docker y Docker Compose en Ubuntu, se pueden seguir los pasos a continuación:
 
1. Para llevar a cabo la instalación, por favor ejecute los siguientes comandos:

```shell

 sudo snap install docker

 sudo apt install docker-compose

```

2. Para verificar la versión de Docker, ejecute el siguiente comando:
 
```shell

 docker-compose --version

```
 
3. Para autorizar a los usuarios a ejecutar comandos Docker, por favor ingrese el siguiente comando:
 
```shell

    sudo groupadd docker

   sudo usermod -aG docker ${USER}

```

4. Asignar los permisos correspondientes al grupo Docker:
 
```shell
    su - ${USER}
```
 
5. Compruebe que el usuario pertenece al grupo Docker:"
 
```shell
    id -nG
```
 
6. Reinicie su equipo para que los cambios surtan efecto.

7. Comandos importantes de Docker para manejo de volumenes

```shell
docker volume ls

docker volume create mivolumen

docker volume rm {nombrevolumen}

docker prune

```

8. Cambiar la contraseña de un contenedor

```shell

docker exec -itu 0 {contenedor} passwd

```


## MongoDB

MongoDB, una de las bases de datos NoSQL más utilizadas y orientadas a documentos, fue tema de discusión en el capítulo anterior. Por lo tanto, no es esencial tener conocimientos previos sobre esta base de datos para comprender el contenido de este capítulo.


Existen diversas maneras de instalar MongoDB, entre ellas:

* Utilizando archivos binarios específicos para tu sistema operativo.  
* Mediante Docker.


### Empleando imágenes de Docker.

Pasos a seguir:

* Ccrear un directorio específico para alojar la base de datos y conceder los permisos de escritura adecuados. A continuación, se presentan los comandos para crear dicho directorio y otorgar los permisos necesarios:

```shell

sudo mkdir -p /data/db

sudo chmod 777 /data/db

sudo mkdir -p /var/log/mongodb

```

* Instalar la ultima versión disponible 

```
docker pull mongodb/mongodb-community-server:latest

docker run -d --name mongodb-latest -p 27017:27017 -v mongodb_data_container:/data/db -d mongodb/mongodb-community-server:latest 


```

* O instalarla implementando autenficación mediante usuario y password

```shell
docker run --name mongodb-lastet -d -p 27017:27017 -e MONGODB_INITDB_ROOT_USERNAME=user -e MONGODB_INITDB_ROOT_PASSWORD=pass -v mongodb_data_container:/data/db -d mongodb/mongodb-community-server:latest 

```


#### Para detener un contenedor
```shell
docker stop mongodb-lastet
```

Para iniciar el contenedor ejecute


```shell
docker start mongodb-lastet
```




### A través de un archivo Docker Compose.


Si prefiere utilizar Docker Compose, sigue estos pasos:

1. Cree un archivo llamado docker-compose.yml.

2. Copie y pegue el siguiente contenido en el archivo docker-compose.yml:
 
```yaml
version: '3.7'
services:
  mongodb_container:
    image: mongo:latest
    ports:
      - 27017:27017
    volumes:
      - mongodb_data_container:/data/db

volumes:
  mongodb_data_container:
  
```




3. Ejecute el comando **docker-compose up** en la terminal dentro del directorio actual para descargar e iniciar el contenedor de MongoDB.
 
```shell
    docker-compose up -d
```

Para detener el contenedor, ejecute el siguiente comando:
  
```shell
    docker-compose stop
```


### Iniciar la imagen de MongoDB

Para iniciar la imagen detenida de MongoDB, siga los siguientes pasos:

1. Navegue hasta el directorio que contiene el archivo docker-compose.yml.

2. Ejecute el siguiente comando en la terminal:
   
```shell
 docker-compose start 
```

## Otra alternativa mediante mongo:noble

```shell
docker pull mongo:noble


docker run -d --name mongodb8.0-noble -p 27017:27017 -v mongodb_data_container:/data/db -d mongo:noble 

```


### Consola de MongoDB


Para administrar MongoDB desde la línea de comandos, siga los siguientes pasos:


1. Identifique el ID del contenedor de la imagen con la que desea interactuar. Puede hacer esto ejecutando el siguiente comando en su terminal:
 
```shell
docker ps -a 
```

2. Para acceder al bash, ejecute el siguiente comando:
 
```shell
docker exec -it mongodb-lastet bash
```
 
3. Crear una carpeta para almacenar copias de seguridad de sus bases de datos:
 
```shell
mkdir home/respados

cd home/respaldos
```
 




 

### MongoDB Shell

**"MongoDB Shell"** es una interfaz interactiva que permite ejecutar comandos directamente en MongoDB. Con ella, podrá crear bases de datos, colecciones, insertar documentos, así como ejecutar consultas y eliminaciones.


Siga los siguientes pasos:

Para comenzar, debe establecer una conexión con la instancia de MongoDB. Esto se puede lograr ejecutando el siguiente comando:

```shell
docker exec -it mongodb-lastet bash
```
 
Inicie **"shell"** de MongoDB ejecutando el comando mongo en su terminal.
 
```shell
mongosh
```

Una vez que la línea de comandos de MongoDB esté en funcionamiento, verá una salida similar a la que se muestra en la siguiente figura:

![]( figura_02_02.png)


Si usted estableció un usuario y password para acceder a la base de datos, debe especificarlo

```shell
mongosh --username user --password pass
```




Para listar todas las bases de datos disponibles en su instancia de MongoDB, utilice el comando show dbs
 
```shell
show dbs
```  

Para crear una nueva base de datos o acceder a una ya existente, utilice el comando use <nombre-basedatos> 
 
```shell
use ejemplodb
```  
 
Para insertar un nuevo documento en una colección de MongoDB, se utiliza el método insertOne(). Este método acepta un documento en formato JSON como argumento, tal como se ilustra en el siguiente ejemplo:

```shell
    db.oceano.insertOne({idoceano:"pacifico", oceano:"Oceano Pacifico"}) 
```  

![]( figura_02_03.png)

El método find() de MongoDB se utiliza para realizar consultas. Por ejemplo, si desea buscar todos los documentos en la colección 'oceano' y visualizar la salida de manera formateada, ejecutaría el siguiente comando:

```shell
  db.oceano.find().pretty() 
```  

La siguiente figura muestra el resultado obtenido al ejecutar el comando find() en MongoDB.


![]( figura_02_04.png)


MongoDB facilita la ejecución de consultas complejas mediante su marco de trabajo de agregación. Posteriormente, se explorará cómo Jmoordbcore gestiona los documentos y las colecciones en MongoDB.


Los métodos insertOne() e insertMany() se utilizan para insertar un solo documento o varios documentos, respectivamente, en las colecciones.

A continuación, se muestra cómo se utilizan estos métodos:

```shell

db.cultura.insertOne(
 {
  "idcultura" : "1", "cultura" : "Americana Antigua"
 }
)


db.persona.insertMany(
 [
    {"idpersona" : "1", "nombre" : "Ana"},
    {"idpersona" : "2", "nombre" : "Maria"}
 ]
)


db.habitante.insertOne(
 {
  "idhabitante" : "1", 
  "persona" : [
    {
      "idpersona" : "1","nombre" : "Ana"
    }, 
    {
      "idpersona" : "2","nombre" : "Maria" 
    }
   ],
   "idioma" :
    {
      "ididioma" : "es","idioma" : "Español",
      "cultura" :
              {
               "idcultura" : "1",
               "cultura" : "Americana" 
               }
    }, 
   "pais" : [
             {
              "idpais" : NumberLong(1),
              "pais" : "Panama Canal" 
             }
      ]
 }
)



db.pais.insertOne(
  { 
    "idpais" : NumberLong(1), "pais" : "Panamá"
  }
)


db.pais.insertOne(
  {
     "idpais" : NumberLong(2), "pais" : "Colombia"
  }
)


db.pais.insertOne(
  { 
    "idpais" : NumberLong(3), "pais" : "Brasil"
  }
)

```

Para cerrar la línea de comandos de MongoDB, simplemente escriba exit y presione la tecla Enter.

```shell
exit
```



### Respaldos  

Para realizar copias de seguridad de la base de datos, utilice el comando mongodump. 

* Siga la instrucción a continuación para almacenar el respaldo en la carpeta respaldo/docker de su sistema.

 
```shell
mongodump --uri=mongodb://127.0.0.1:27017 -d ejemplodb -o /home/respaldo/ejemplodb
```

Si desea crear un respaldo de la base de datos y guardarlo como un archivo comprimido, puede hacerlo utilizando el siguiente comando: 

```shell
mongodump --archive=test.ejemplo.gz --gzip --db=ejemplodb

```

* Si establecio seguridad para la base de datos, debe indicar las credenciales


```shell
mongodump --uri=mongodb://127.0.0.1:27017 -d ejemplodb -o /home/respaldo/ejemplodb --username user --password pass
```

Si desea crear un respaldo de la base de datos y guardarlo como un archivo comprimido, puede hacerlo utilizando el siguiente comando: 

```shell
mongodump --archive=test.ejemplo.gz --gzip --db=ejemplodb --username user --password pass

```



Finalmente, para copiar el archivo de respaldo desde el contenedor Docker a su sistema operativo local, abra la consola y ejecute el siguiente comando:
 
```shell
docker cp mongodb-lastet:/home/respaldo/ejemplodb  /home/respaldo/ejemplodb

```
 


### Restauración 

La restauración de una base de datos MongoDB, se utiliza el comando mongorestore. Por ejemplo, para restaurar la base de datos desde un respaldo almacenado en la ruta
 
```shell
mongorestore --uri=mongodb://127.0.0.1:27017  /home/respaldo/ejemplodb
```
 
Además, si la base de datos fue respaldada en un archivo gzip, puede restaurarla utilizando el siguiente comando:


```shell
mongorestore --gzip --archive=ejemplodb.gz 
```

* Si establecio seguridad para la base de datos, debe indicar las credenciales

```shell
mongorestore --uri=mongodb://127.0.0.1:27017  /home/respaldo/ejemplodb --username user --password pass
```
 
Además, si la base de datos fue respaldada en un archivo gzip, puede restaurarla utilizando el siguiente comando:


```shell
mongorestore --gzip --archive=ejemplodb.gz  --username user --password pass
```


### Copiar archivos  

Docker ofrece una manera fácil de copiar archivos utilizando el comando **cp**. Por ejemplo, para copiar el archivo 'ejemplodb' desde el contenedor Docker a la carpeta '/home/respaldo/Descargas' en su sistema local, ejecutaría el siguiente comando:
 
```shell
docker cp e321ee10e65e:/home/respaldo/ejemplodb /home/respaldo/Descargas/ejemplodb
```

Si desea copiar un archivo desde su sistema local al contenedor Docker, simplemente invierta el orden de los argumentos de origen y destino:
 
```shell
docker cp /home/respaldo/Descargas/ejemplodb e321ee10e65e:/home/respaldo/ejemplodb 
```

Después de familiarizarnos con MongoDB y Docker, nuestro próximo paso será crear un proyecto Jakarta EE. Sin embargo, a diferencia del capítulo 1, en este capítulo utilizaremos Apache NetBeans IDE para la configuración del proyecto.

## Generar proyecto Jakarta EE

Para generar un proyecto Jakarta EE, existen varias opciones.  En este libro, nos centraremos en algunas de ellas:

* Payara Starter

* Jakarta EE Starter

* Arquetipo Maven

* Microprofile Starter

* Apache NetBeans IDE

En las siguientes secciones, se detalla cómo crear un proyecto Jakarta EE utilizando cada una de estas opciones:



### Payara Starter

En el capítulo anterior, ya demostramos cómo crear un proyecto utilizando Payara Starter. Por lo tanto, no repetiremos ese proceso en este capítulo.


### JakartaEE Starter

Para generar un proyecto Web con JakartaEE, visite la página [https://start.jakarta.ee/](https://start.jakarta.ee/). Luego, proporcione los detalles del proyecto que desea crear:

* Versión de Jakarta: **Jakarta EE 10**

* Perfil de Jakarta EE: **Platform**

* Versión de Java SE: **Java SE 17** 

* Runtime: **Payara**

* Soporte para Docker: **Yes**


Finalmente, haga clic en el botón Generate para iniciar la descarga del archivo jakartaee-hello-world.zip, que contiene el proyecto Jakarta EE.


Ahora vamos a crearlo utilizando un arquetipo maven.


### Arquetipo Maven

Según el sitio web oficial de Maven 

[https://maven.apache.org/guides/introduction/introduction-to-archetypes.html](https://maven.apache.org/guides/introduction/introduction-to-archetypes.html), 


**"Un arquetipo es un conjunto de plantillas de herramientas para proyectos Maven. Se define como un patrón o modelo original del cual se crean todas las demás cosas del mismo tipo."**

Para generar un nuevo proyecto utilizando el arquetipo de Maven, ejecute el siguiente comando en la consola:

```
mvn archetype:generate -DarchetypeGroupId=org.eclipse.starter -DarchetypeArtifactId=jakartaee10-minimal -DarchetypeVersion=1.0.0 -DgroupId=com.jmoordbcore -DartifactId=capitulo02 -Dprofile=api -Dversion=1.0.0-SNAPSHOT -DinteractiveMode=false

```

Una vez que se complete la ejecución del comando, se creará un directorio que contiene la estructura del proyecto Jakarta EE.


A continuación, vamos a crear el proyecto utilizando Microprofile Starter.

### Microprofile Starter

Microprofile Starter [https://start.microprofile.io/](https://start.microprofile.io/), es una herramienta en línea que facilita la creación de proyectos Microprofile. Esta herramienta permite seleccionar las APIs de Microprofile que desea incluir en su proyecto.

En los capítulos siguientes, proporcionaremos ejemplos detallados sobre cómo crear un proyecto utilizando Microprofile Starter.

A continuación, vamos a crear el proyecto utilizando Apache NetBeans IDE.

#### Apache NetBeans IDE

Inicie Apache NetBeans IDE y siga estos pasos para crear un nuevo proyecto:

1. Elija **New Project** desde el menú File.

2. En la sección Categories, seleccione **Java with Maven**. Posteriormente, en la sección Project, elija **Web Application**



![]( figura_02_05.png)


Al hacer clic en el botón Next, se abrirá un cuadro de diálogo solicitando detalles como el nombre del proyecto, la ubicación de almacenamiento y la configuración del artefacto Maven.

Los detalles del artefacto incluyen: 


* artifactId: Identificador único para el proyecto.  

* groupId: Identifica la organización o grupo que mantiene el proyecto.

* version: Número de versión del proyecto, que debe ser incrementable. 

* package: Define el paquete principal del proyecto.
  


![]( figura_02_06.png)


Haga clic en el botón **Next**, se abrirá un cuadro de diálogo que le permitirá seleccionar el servidor y la versión de JakartaEE para su proyecto.

Es aconsejable optar por la versión más reciente de la especificación para mantener su proyecto al día.

En el cuadro de diálogo  Server, elija **No Server Selected** y para Java EE Version, seleccione  **JakartaEE 10 Web**.


Verá una lista de servidores que ha configurado previamente en NetBeans IDE.

Para este proyecto, utilizaremos Payara Micro, por lo que la opción del servidor debe mantenerse en No Server Selected. Realizaremos todas las configuraciones directamente en el archivo pom.xml.

![]( figura_02_07.png)



### PayaraMicro


Según su sitio web oficial[https://www.payara.fish/learn/getting-started-with-payara-micro/](https://www.payara.fish/learn/getting-started-with-payara-micro/), Payara Micro es una plataforma de código abierto diseñada para implementar aplicaciones y microservicios JakartaEE en contenedores. Se destaca por su tamaño compacto y su facilidad de uso, ya que no requiere instalaciones ni configuraciones complejas.


Vamos a convertir el proyecto Web a Payara Micro utilizando el plugin integrado en NetBeans IDE.


 Desde el menú, seleccione File y luego New File. En el asistente que aparece, diríjase a la sección Categories y seleccione **Payara**. Luego en File Types, elija  **Payara Micro Maven plugin**.
  
![]( figura_02_08.png)

Una vez que haga clic en el botón Siguiente, se le pedirá que elija la versión del plugin. Asegúrese de seleccionar la versión más reciente. Puede comprobar la última versión disponible de Payara Micro Community Edition en su página oficial [https://www.payara.fish/downloads/payara-platform-community-edition/](https://www.payara.fish/downloads/payara-platform-community-edition/).

![](figura_02_09.png)

Una vez que el proceso se complete, notará que el icono del proyecto ha cambiado. Haga clic con el botón derecho en el proyecto y cambie el nombre de capitulo02-1.0-SNAPSHOT a simplemente capitulo02.

![]( figura_02_10.png)

Revise la configuración especificada para el plugin Payara Micro en abriendo el archivo pom.xml. Este plugin ofrece varias opciones de configuración, como la posibilidad de mostrar el logo de Payara al ejecutar la aplicación, habilitar o deshabilitar Hazelcast, y agregar parámetros para la JVM, tal como se muestra en la siguiente sección:

```xml

<plugin>
 <groupId>fish.payara.maven.plugins</groupId>
 <artifactId>payara-micro-maven-plugin</artifactId>
  <configuration>
    <payaraVersion>${version.payara}</payaraVersion>
    <deployWar>false</deployWar>
    <commandLineOptions>
       <option>
         <key>--autoBindHttp</key>
       </option>
       <!-- desabilita Hazelcas -->
       <option>
          <key>--noHazelcast</key>
       </option>
       <option>
         <key>--logo</key>
       </option>
       <option>
       <key>--deploy</key>
       <value>${project.build.directory}/${project.build.finalName}</value>
      </option>
    </commandLineOptions>
<!--
JDK 17+ Soluciona error con EJB
-->                         
    <javaCommandLineOptions>
       <option>
        <key>--add-opens</key>
        <value>java.base/java.io=ALL-UNNAMED</value>
       </option>
       <option>
        <key></key>
        <value>-Djdk.util.zip.disableZip64ExtraFieldValidation=true</value>
       </option>
    </javaCommandLineOptions>    
   </configuration>
  <version>2.0</version>
</plugin>

``` 


Por favor, realice las siguientes configuraciones en el archivo pom.xml:


1. Establezca la versión de Jakarta EE a 10.0.0.

```xml

 <jakartaee>10.0.0</jakartaee>

```

2. Añada la propiedad final.name


```xml
    <final.name>capitulo02</final.name>

```

3. Actualice la versión de source a 21 o superior.

4. Asegúrese de que todas las versiones de los plugins de Maven estén actualizadas a la última versión disponible.

5. Incluya la dependencia de Microprofile en su proyecto.

6. Defina las propiedades necesarias para especificar las versiones de Payara que son compatibles con JakartaEE 10, microprofile-config, microprofile-metrics, microprofile-health y jmoordbcore.

A continuación, se presentan las propiedades que estaban disponibles al momento de escribir este libro. Sin embargo, siempre debe utilizar las versiones más actualizadas:

```xml

<microprofile.version>6.1</microprofile.version>
<version.payara>6.2024.12</version.payara>
<microprofile-config-api.version>3.1</microprofile-config-api.version>
<microprofile-metrics-api.version>5.1.0</microprofile-metrics-api.version>
<microprofile-health-api.version>4.0.1</microprofile-health-api.version>
<version.jmoordbcore>2.0.2</version.jmoordbcore>
<jakartaee>10.0.0</jakartaee>

```
 
7. Establezca las dependencias de Maven en su archivo POM, siguiendo el formato proporcionado en el siguiente fragmento de código:

```xml
 
<dependencies>
   <dependency>
      <groupId>jakarta.platform</groupId>
      <artifactId>jakarta.jakartaee-api</artifactId>
      <version>${jakartaee}</version>
      <scope>provided</scope>
   </dependency>

   <dependency>
      <groupId>org.eclipse.microprofile</groupId>
      <artifactId>microprofile</artifactId>
      <version>${microprofile.version}</version>
      <type>pom</type>
      <scope>provided</scope>
   </dependency>

   <dependency>
      <groupId>org.eclipse.microprofile.config</groupId>
      <artifactId>microprofile-config-api</artifactId>
      <version>${microprofile-config-api.version}</version>
   </dependency>

   <dependency>
      <groupId>org.eclipse.microprofile.metrics</groupId>
      <artifactId>microprofile-metrics-api</artifactId>
      <version>${microprofile-metrics-api.version}</version>
   </dependency>
   <dependency>
      <groupId>org.eclipse.microprofile.health</groupId>
      <artifactId>microprofile-health-api</artifactId>
      <version>${microprofile-health-api.version}</version>
      <type>jar</type>
   </dependency>
 
   <dependency>
      <groupId>com.github.avbravo</groupId>
      <artifactId>jmoordb-core</artifactId>
      <version>${version.jmoordbcore}</version>
   </dependency>
</dependencies>
 


```


8. Incluya el repositorio jitpack.io en su configuración.
 
```xml

<repositories>
  <repository>
    <id>jitpack.io</id>
    <url>https://jitpack.io</url>
  </repository>
</repositories>

```

Consulte el archivo pom.xml del proyecto 'capitulo2' para obtener una visión detallada de su configuración.

Vamos a proporcionar una descripción general de las APIs que estamos utilizando.


### Jakarta RESTful Web Services

[Jakarta RESTful Web Services](https://jakarta.ee/specifications/restful-ws/3.1/jakarta-restful-ws-spec-3.1) ofrece un conjunto de APIs Java que facilitan el desarrollo de servicios Web basados en la arquitectura REST(Representational State Transfer).
 

Durante la creación del proyecto, la ruta principal del endpoint se configuró en la clase JakartaRestConfiguration. Para modificar esta ruta, cambie el valor de la anotación @ApplicationPath de resources a api.

Proceda a realizar el cambio

```java
@ApplicationPath("resources")
```

Por

```java
@ApplicationPath("api")
```

Al modificar el URI de consulta, la nueva dirección sería [http://localhost:8080/capitulo02/api](http://localhost:8080/capitulo02/api)


### OpenAPI

La especificación [Microprofile OpenAPI](https://download.eclipse.org/microprofile/microprofile-open-api-3.1/microprofile-openapi-spec-3.1.html) ofrece un conjunto de APIs Java que implementan la especificación [OpenAPI](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.0.0.md). Esto permite a las personas y a los sistemas informáticos descubrir las API RESTful sin la necesidad de acceder al código fuente.


La configuración se gestiona a través del archivo **microprofile-config.properties**. Los detalles de estas configuraciones son implementadas por los vendedores (empresas que ofrecen productos compatibles con las especificaciones):

* mp.openapi.model.reader   

* mp.openapi.filter
 
* mp.openapi.scan.disable

* mp.openapi.scan.packages

* mp.openapi.scan.classes

* mp.openapi.scan.exclude.packages

* mp.openapi.scan.exclude.classes

* mp.openapi.servers

* mp.openapi.servers.path

* mp.openapi.servers.operation


#### Mecanismos de Documentación

La documentación se facilita mediante el uso de anotaciones proporcionadas por Microprofile OpenAPI. El URI de salida para acceder a esta documentación será /openapi. 

Las anotaciones disponibles en Microprofile OpenAPI incluyen:

```
@Callback             @Callbacks               @CallbackOperation    
@Components           @Explode                 @ParameterIn  
@ParameterStyle       @SecuritySchemeIn        @SecuritySchemeType
@Extension            @Extensions              @ExternalDocumentation
@Header               @Contact                 @Info                 
@License              @Link                    @LinkParameter
@Content              @DiscriminatorMapping    @Encoding
@ExampleObject        @Schema                  @OpenAPIDefinition
@Operation            @Parameter               @Parameters
@RequestBody          @APIResponse             @APIResponses 
@OAuthFlow            @OAuthFlows              @OAuthScope      
@SecurityRequirement  @SecurityRequirements    @SecurityRequirementsSet
@SecurityScheme


```


A continuación, se proporciona una descripción general de estas anotaciones y cómo se utilizan.

#### @OpenAPIDefinition

Use esta anotación para establecer metadatos generales para su API OpenAPI, como el título, el autor, el correo electrónico y la información de la versión del servidor. Puede hacer esto modificando la clase JakartaRestConfiguration e incluyendo la anotación @OpenApiDefinition.

Por ejemplo:

```java


import jakarta.ws.rs.ApplicationPath;
import jakarta.ws.rs.core.Application;
import org.eclipse.microprofile.openapi.annotations.OpenAPIDefinition;
import org.eclipse.microprofile.openapi.annotations.info.Contact;
import org.eclipse.microprofile.openapi.annotations.info.Info;
import org.eclipse.microprofile.openapi.annotations.servers.Server;


@ApplicationPath("api")
@OpenAPIDefinition(info = @Info(
       title = "Jmoordbcore",
       description = "capitulo 02",
       version = "1.0.0-Snapshot",
       contact = @Contact(
               name = "AVbravo",
               email = "avbravo@gmail.com",
               url = "https://avbravo.blogspot.com")
       ),
       servers = {
          @Server(url = "http://localhost:8080/", description = "Servidor Local "),}
)

public class JakartaRestConfiguration extends Application {
    
}

```

Inicie el proyecto utilizando los siguientes comandos Maven en la consola:

```shell
mvn clean verify
mvn payara-micro:start


```

Abra su navegador Web e ingrese la dirección URL [http://localhost:8080/openapi](http://localhost:8080/openapi). Esta URL le llevará a la documentación generada por OpenAPI:

```
openapi: 3.0.0
info:
  title: Jmoordbcore
  description: capitulo 02
  contact:
    name: AVbravo
    url: https://avbravo.blogspot.com
    email: avbravo@gmail.com
  version: 1.0.0-Snapshot
servers:
- url: http://localhost:8080/
  description: 'Servidor Local '

```

Para detener la ejecución del programa, presione las teclas CTRL+C.


#### @Operation

La anotación @Operation se emplea para proporcionar una descripción detallada de una operación específica en un servicio Jakarta RESTful Web Services.


Por ejemplo, en el caso del método findAll(), esta anotación permite explicar su funcionalidad de manera exhaustiva.

```java

@GET
@Path("/findAll")
@Operation(summary = "Listado de paises",
           description = "Devuelve la lista de todos los paises")
public List<Pais> findAll() {

}

```

Para ejecutar el proyecto, utilice los siguientes comandos Maven en la consola:

```shell
mvn clean verify

mvn payara-micro:start


```


Al consultar [http://localhost:8080/openapi](http://localhost:8080/openapi), se obtendría una salida similar a la siguiente:

``````

/pais/findAll
  get:
    summary: Listado de paises
    description: Devuelve la lista de todos los paises
    operationId: findAll

```

Para detener la ejecución, presione las teclas CTRL+C.


#### @ApiResponse

La anotación @ApiResponse se utiliza para especificar una respuesta individual de una operación. A menudo se emplea para describir las diversas respuestas que una operación podría devolver. Por ejemplo:

```java

@GET
@Path("/findAll")
@Operation(summary = "Obtiene todos los paises", description = "Retorna todos los paises disponibles")
   @APIResponse(responseCode = "500", description = "Servidor inalcanzable")
   @APIResponse(responseCode = "200", description = "Los paises")


public List<Pais> findAll() {
 }
```

Ejecute el proyecto utilizando los siguientes comandos Maven en la consola:

```shell
mvn clean verify
mvn payara-micro:start


```
Una vez que el proyecto esté en ejecución, puede consultar la documentación en el siguiente enlace [http://localhost:8080/openapi](http://localhost:8080/openapi). 

Esto devolverá una respuesta similar a la siguiente:

```
/api/pais:
  get:
    tags:
    - BETA
     summary: Obtiene todos los paises
     description: Retorna todos los paises disponibles
     operationId: findAll
     responses:
       "200":
         content:
           application/json:
             schema:
               exclusiveMaximum: false
               exclusiveMinimum: false
               minLength: 0
               uniqueItems: false
               maxProperties: 0
               minProperties: 0
               type: array
               description: los paises
               nullable: false
               readOnly: true
               writeOnly: false
               deprecated: false
               items:
                 type: array
         description: Los paises
       "500":
         description: Servidor inalcanzable
     deprecated: false

```


Como puede ver, el uso de Microprofile OpenAPI es bastante sencillo y solo requiere el uso de anotaciones. 

Para detener la ejecución del programa, presione las teclas CTRL+C.



### Microprofile-Config

[Microprofile Config](https://download.eclipse.org/microprofile/microprofile-config-3.0/microprofile-config-spec-3.0.html) ofrece una API simple para administrar la configuración de las aplicaciones.

Las configuraciones se establecen en un archivo denominado microprofile-config.properties, al cual se puede acceder posteriormente a través de la API.

Este archivo se encuentra en el directorio META-INF, como se muestra en la figura siguiente.


![]( figura_02_11.png)


Para utilizar Jmoordbcore, es necesario configurar ciertos parámetros en el archivo microprofile-config.properties:

|Atributo          | Descripción                                                | 
|-------------------       | -------------------------------------------------------    |
|                  |                                                            |
|mongodb.uri       |URL de la base de datos.                      |
|mongodb.database  |Nombre de la base de datos.                                  |            
|mongodb.database# |Número de bases de datos.    |            
|mongodb.jmoordb   |Base de datos de configuración para  jmoordbcore.  |            



 mongodb.uri se configura de tres maneras:

* Conexión a MongoDB Atlas

```java
mongodb.uri=mongodb+srv://mongodb/?retryWrites=true&w=majority;
```

* Conexión a MongoDB Local o Docker sin autentificación

```java
mongodb.uri=mongodb://localhost:27017
```

* Conexión a MongoDB Local o Docker con autentificación

```java
mongodb.uri=mongodb://user:password@localhost:27017
```



En este ejemplo, se utiliza la propiedad mongodb.database, mongodb.database1 y mongodb.database2 para referirse a la base de datos predeterminada, base de datos 1 y base de datos 2 respectivamente.
 


```
#mongodb.uri=mongodb+srv://mongodb/?retryWrites=true&w=majority;
#---------------------------------------
# MongoDB con seguridad user y password
#mongodb.uri=mongodb://user:password@localhost:27017
#---------------------------------------
mongodb.uri=mongodb://localhost:27017
#-- Base de datos de configuración jmoordb
mongodb.jmoordb= configurationjmoordbdb
#-- Base de datos
mongodb.database=ejemplodb
mongodb.database1=testdb
mongodb.database2=practicadb
```


![]( figura_02_12.png)


Para ejecutar el proyecto, utilice los siguientes comandos Maven:

```shell
mvn clean verify

mvn payara-micro:start

```

Una vez que el proyecto esté en ejecución, puede acceder a él a través de su navegador web preferido utilizando la dirección [http://localhost:8080/capitulo02/](http://localhost:8080/capitulo02/).

Se mostrará un mensaje de bienvenida. 

Para detener la ejecución del proyecto, presione las teclas CTRL+C.

![]( figura_02_13.png)


#### UberJar

Un UberJar es un archivo .jar que empaqueta la aplicación, su configuración y dependencias.

Para generar un UberJar con Payara Micro, sigue estos pasos:

1. Limpie el proyecto y ejecute las pruebas con el siguiente comando:

```shell

mvn clean verify

```

2. Ejecuta el siguiente comando en la consola:

```shell

mvn payara-micro:bundle

```

Una vez finalizado, se creará el archivo capitulo02-microbundle.jar en la carpeta target de su proyecto.

3. Ejecute el archivo con el comando java -jar. También puede agregar la configuración de Payara Micro si lo necesita.


```shell

java -jar  target/capitulo02-microbundle.jar --noHazelcast

```

La opción --noHazelcast desactiva Hazelcast, que es un sistema de caché utilizado por Payara Micro.


Accede a la aplicación utilizando el siguiente URL [http://localhost:8080](http://localhost:8080).

Después de revisar de manera general algunos conceptos sobre la creación de microservicios con Microprofile y JakartaEE usando Payara Micro, ahora nos enfocaremos en la configuración requerida por Jmoordbcore.



## Configuración de Jmoordbcore

En las siguientes secciones, aprenderá a configurar su proyecto para interactuar con una base de datos MongoDB usando Jmoordbcore.



#### MongoDBProducer

Para manejar las conexiones a MongoDB, usaremos una clase Java denominada MongoDBProducer. Esta clase nos permitirá inyectar un objeto MongoClient, que contiene la conexión a la base de datos, directamente en los repositorios. Profundizaremos en el concepto de repositorio más adelante.

En el proyecto, cree un nuevo paquete llamado producer. Añada una nueva clase Java llamada MongoDBProducer.java, como se muestra en el siguiente segmento de código:

 
```java

import com.jmoordb.core.annotation.DateSupport;
import com.jmoordb.core.annotation.enumerations.JakartaSource;
import com.mongodb.client.MongoClient;
import com.mongodb.client.MongoClients;
import jakarta.enterprise.context.ApplicationScoped;
import jakarta.enterprise.inject.Disposes;
import jakarta.enterprise.inject.Produces;
import jakarta.inject.Inject;
import java.io.Serializable;
import org.eclipse.microprofile.config.Config;
import org.eclipse.microprofile.config.inject.ConfigProperty;

@ApplicationScoped
@DateSupport(jakartaSource = JakartaSource.JAKARTA)
public class MongoDBProducer implements Serializable {

   @Inject
   private Config config;
   @Inject
   @ConfigProperty(name = "mongodb.uri")
   private String mongodburi;
    
   @Produces
   @ApplicationScoped
   public MongoClient mongoClient() {
       MongoClient mongoClient = MongoClients.create(mongodburi);
      return mongoClient;

   }

   public void close(@Disposes final MongoClient mongoClient) {
       mongoClient.close();
   }

}

```


Este código no varía entre proyectos y debería utilizarse tal cual. En él, puede observar que se utiliza la anotación @Produces con alcance ApplicationScoped, lo que permite su uso en toda la aplicación. Estas anotaciones toman el valor de mongodb.uri, que se crea en el archivo microprofile-config.properties mediante la API Microprofile Config.

Al usar @Inject MongoClient en las implementaciones de los repositorios, como se muestra a continuación:

```java

    @Inject
    MongoClient mongoClient;

```

Se invoca automáticamente el método public MongoClient mongoClient() que devuelve la conexión a MongoDB. Jmoordbcore se encarga de esto automáticamente, solo necesitas definir la clase MongoDBProducer de manera similar al código mostrado.

 

## Entidad (Entity)

Una entidad (Entity), fue definida por [Martin Fowler](https://martinfowler.com/bliki/EvansClassification.html) como
**Objetos que tienen una identidad propia que atraviesa el tiempo y diferentes representaciones. Una entidad se utiliza a menudo para representar una tabla de base de datos o un objeto que corresponde a una tabla de base de datos**.


En JmoordbCore, una entidad (Entity) es una clase Java que representa un documento en una colección MongoDB.

La anotación @Entity se utiliza para marcar una entidad. @Id se emplea para definir la llave primaria, mientras que @Column se utiliza para especificar las columnas.

Para crear una entidad en JmoordbCore, cree un paquete llamado 'model' y añada una clase, como por ejemplo Oceano:

```java

@Entity(jakartaSource = JakartaSource.JAKARTA)
public class Oceano {
    @Id
    private String idoceano;
    @Column
    private String oceano;


    public Oceano() {
    }
//set/get
}

```


La entidad Oceano, representa un documento en una colección de MongoDB llamada oceano, de la siguiente manera:

```json

{
"idoceano":"pacifico",
"oceano": "Pacifico"
}

```

 
### Repositorio (Repository)

El patrón repositorio establece que mediante una interfaz mediara entre el dominio y las capas de mapeo de datos.

Para crear un repositorio en Jmoordbcore, necesitas crear una interfaz Java y decorarla con la anotación @Repository. Esto permitirá a Jmoordbcore generar las clases y métodos necesarios para interactuar con MongoDB durante la compilación.

Los conceptos fundamentales para crear repositorios se explican en el Capítulo 4. A continuación, se muestra un ejemplo de cómo se vería esto en código:


```java

@Repository(entity = Oceano.class)
public interface OceanoRepository extends CrudRepository<Oceano, String> {
    
}

```
 
Al compilar el proyecto, se generarán las clases y métodos necesarios para interactuar con la base de datos. Los detalles sobre las anotaciones necesarias para crear entidades y repositorios se explicarán en los capítulos siguientes.


![]( figura_02_14.png).


## Autoincrementable

MongoDB no ofrece soporte para valores autoincrementables o secuenciales. Sin embargo, Jmoordbcore implementa esta funcionalidad utilizando colecciones, lo cual se explicará en los capítulos siguientes.

Jmoordbcore necesita que se defina la interfaz AutogeneratedRepository.java en el paquete 'repository'. Esta interfaz se usará para manejar valores incrementales o secuenciales.

Para una explicación detallada de su uso, consulta el Capítulo 5.
 
```java


import com.jmoordb.core.annotation.autosecuence.Autogenerated;
import com.jmoordb.core.annotation.autosecuence.AutosecuenceRepository;
import com.jmoordb.core.annotation.enumerations.JakartaSource;
import com.jmoordb.core.model.Autosequence;


@AutosecuenceRepository(entity = Autosequence.class)
public interface AutogeneratedRepository {


    @Autogenerated()
    public Long generate(String database, String collection);


}


```

#### Jakarta RESTful Web Services

Usando Jakarta RESTful Web Services, generaremos el endpoint para la colección 'Oceano'. Cree una clase llamada 'OceanoController' en el paquete 'controller'.

A continuación, se muestra un fragmento de código de la clase OceanoController:

```java


import com.jmoordbcore.capitulo02.model.Oceano;
import com.jmoordbcore.capitulo02.repository.OceanoRepository;
import jakarta.inject.Inject;
import jakarta.ws.rs.GET;
import jakarta.ws.rs.Path;
import jakarta.ws.rs.Produces;
import jakarta.ws.rs.core.MediaType;
import java.util.List;


/**
 *
 * @author avbravo
 */
@Path("oceano")
public class OceanoController {
    @Inject
    OceanoRepository oceanoRepository;
    
    @GET
    @Produces(MediaType.APPLICATION_JSON)
    public List<Oceano> findAll(){
        return oceanoRepository.findAll();
    }
    
  
}


```

Ejecuta el proyecto con los siguientes comandos Maven en la consola:

```shell

mvn clean verify payara-micro:start

```


Para acceder a la información del océano, puedes hacerlo a través de tu navegador web, Postman o, Curl, consultando la siguiente dirección [http://localhost:8080/capitulo02/api/oceano](http://localhost:8080/capitulo02/api/oceano).


![](figura_02_15.png)

Para empaquetar y ejecutar el proyecto, usa los siguientes comandos Maven:


```shell

mvn clean verify payara-micro:bundle


java -jar target/capitulo02-m*.jar --noHazelcast


```


Acceda al URL [http://localhost:8080/api/oceano](http://localhost:8080/api/oceano) desde el navegador Web, Postman o Curl.




## Bases de datos y colecciones dinámicas

Para crear o asignar dinámicamente una base de datos en tiempo de ejecución, debe utilizar el método **setDynamicDatabase("database")**. 


Si desea generar dinámicamente una colección en tiempo de ejecución utilice el método **setDynamicCollection("");**


Consideraciones importantes:

* El nombre de base de datos tiene precedencia sobre el nombre de base de datos establecido en el archivo microprofile-config.properties. 

* El nombre de colección tiene precedencia sobre el nombre de base de datos establecido en el archivo microprofile-config.properties. 

* La creación dinámica de bases de datos y colecciones tiene la restricción de aplicar solo a entidades que no utilicen referencias.


Ejemplo de uso de bases de datos dinámicas

```java


import com.jmoordbcore.capitulo02.model.Oceano;
import com.jmoordbcore.capitulo02.repository.OceanoRepository;
import jakarta.inject.Inject;
import jakarta.ws.rs.GET;
import jakarta.ws.rs.Path;
import jakarta.ws.rs.Produces;
import jakarta.ws.rs.core.MediaType;
import java.util.List;


/**
 *
 * @author avbravo
 */
@Path("oceano")
public class OceanoController {
    @Inject
    OceanoRepository oceanoRepository;
    
    @GET
    @Produces(MediaType.APPLICATION_JSON)
    public List<Oceano> findAll(){
           oceanoRepository.setDynamicDatabase("database_name")
           oceanoRepository.setDynamicCollection("coleccion_name");
           return oceanoRepository.findAll();
    }
    
  
}
```


## Resumen

En este capítulo, hemos introducido Jmoordbcore y sus requisitos mínimos, incluyendo la instalación de Docker para trabajar con imágenes de MongoDB. Hemos creado un proyecto Jakarta EE con NetBeans IDE para definir una entidad, configurar la base de datos y crear un repositorio.

En el próximo capítulo, profundizaremos en las entidades y sus anotaciones para declarar documentos embebidos y referenciados.
