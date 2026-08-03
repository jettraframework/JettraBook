# Capítulo 1

Este capítulo representa una introducción a las especificaciones de Jakarta EE y Eclipse Microprofile para la creación de aplicaciones Java orientadas a la nube.

Este capítulo incluye los temas:

* Jakarta EE

* Aplicación Jakarta EE 

* Microprofile

* NoSQL

* MongoDB


## Introducción  

Jmoordbcore es un marco de trabajo Java para bases de datos NoSQL, que utiliza las APIs de Jakarta EE y Microprofile.

Desde los primeros años, el lenguaje y plataforma Java se han consolidado a nivel empresarial. En la actualidad, Java es líder en la creación de aplicaciones en la nube computacional.

En este capítulo haremos un breve recorrido por las especificaciones Jakarta EE, abordando conceptos del lenguaje de programación Java. Adicional se presenta una guía básica de algunos conceptos esenciales para crear un marco de trabajo Java para bases de datos NoSQL.


## Jakarta EE

[Jakarta EE ](https://jakarta.ee/), es un conjunto de especificaciones Java, para el entorno empresarial que han surgido de la evolución de Java EE.



Las implementaciones de las especificaciones de Jakarta facilitan la creación de aplicaciones al permitir el uso de las mejores prácticas de desarrollo.

El listado de las especificaciones se muestra a continuación:



|Especificación                            | Especificación               | 
|-----------                               |------------------------------|
|                                          |                              |     
|Jakarta EE Platform                       | Jakarta EE Web Profile       |
|Jakarta Activation                        | Jakarta Annotations          |
|Jakarta Authorization                     | Jakarta Batch                |
|Jakarta Concurrency                       | Jakarta Config               |
|Jakarta Contexts and Dependency Injection | Jakarta Data                 |
|Jakarta Dependency Injection              | Jakarta Deployment           |
|Jakarta Enterprise Web Services           | Jakarta Expression Language  |
|Jakarta Interceptors                      | Jakarta JSON Binding         |
|Jakarta Mail                              | Jakarta Managed Beans        |
|Jakarta Messaging                         | Jakarta MVC                  |
|Jakarta Persistence                       | Jakarta RESTful Web Services |
|Jakarta Security                          | Jakarta Server Pages         |
|Jakarta SOAP with Attachments             | Jakarta Standard Tag Library |
|Jakarta Web Services Metadata             | Jakarta WebSocket            |
|Jakarta XML Registries                    | Jakarta XML RPC              |
|Jakarta EE Core Profile                   | Jakarta Authentication       |
|Jakarta Bean Validation                   | Jakarta Connectors           |
|Jakarta Debugging Support for Other Languages | Jakarta Enterprise Beans     |
|Jakarta Faces                             | Jakarta JSON Processing      | 
|Jakarta NoSQL                            | Jakarta RPC                  | 
|Jakarta Servlet                          | Jakarta XML Binding          |
|Jakarta XML Web Services Specification    | Jakarta Management           |
|Jakarta Transactions                      |                              |

Si desea conocer un poco más de Jakarta EE , junto a [Geovany Mendoza](https://twitter.com/geovanny0401) y [Otávio Gonçalves de Santana](https://twitter.com/otaviojava/) escribimos el libro [Building Modern Web Applications With Jakarta EE, NoSQL Databases and Microservices: Create Web Applications Jakarta EE with Microservice](https://www.amazon.co.uk/Building-Applications-Jakarta-Databases-Microservices/dp/9389423341) .



## Microprofile

[Microprofile](http://microprofile.io) es un conjunto de especificaciones Java diseñadas para la creación de microservicios. A lo largo de los capítulos de este libro, se generarán ejemplos que ilustran la implementación de estas especificaciones.
A continuación, se presenta la lista de las especificaciones de Eclipse Microprofile:

|Especificación    | Especificación |
|-----------       | -----------    |
|OpenTelemetry     | OpenAPI        |
|Rest Client           | Config|
|Fault Tolerance         |Metrics           |
|JWT Authentication          | Health        |
|Jakarta EE 10 Core Profile    |  |


## Aplicación Jakarta EE 

En la sección siguiente, vamos a generar una aplicación Jakarta. 

Los requisitos para este capítulo son: 

### Java

Visite el sitio de Adoptium [https://adoptium.net/es/](https://adoptium.net/es/), para descargar la versión del OpenJDK de su preferencia. Siga las instrucciones para instalarlo en su sistema operativo.

### Apache Maven. 

Según su definición oficial en su sitio web [https://maven.apache.org/](https://maven.apache.org/), Apache Maven es una herramienta de gestión y comprensión de proyectos de software. Basado en el concepto de modelo de objetos de proyecto (POM), Maven puede gestionar la compilación, los informes y la documentación de un proyecto a partir de una pieza central de información. 

Maven utiliza el archivo XML llamado pom.xml, acrónimo de Project Object Model. En este archivo se especifican las dependencias, configuraciones e instrucciones necesarias para la construcción del software.


Para crear el proyecto utilizando un arquetipo de Maven, necesitará  proporcionar la siguiente información:

* groupId: Identifica a la organización o grupo de desarrollo.

* artifactId: Identifica el paquete dentro del grupo de desarrollo.

* version: Indica la versión del artefacto.

* package: Define el paquete principal.

Este tutorial, disponible en [https://github.com/jfrchicanog/maven-tutorial](https://github.com/jfrchicanog/maven-tutorial), proporciona información importante sobre Maven que debe conocer.

Puede descargar el archivo binario de Maven para su sistema operativo y configurar las variables de entorno correspondiente.


* Verificar la instalación

Una vez instalado los requisitos, puede verificar la instalación de Java y Maven mediante los comandos:

```shell
java -version

mvn -version


```

La consola mostrará las versiones de Java y Maven que has instalado. Si no se muestran, por favor revisa el proceso de instalación y configuración.

Ejecutaremos varias fases en Maven entre ellas:

* clean: Limpiar el proyecto.

* verify: Comprueba que las pruebas de integración sean pasadas correctamente.

* payara-micro:start: Ejecuta el proyecto sobre Payara Micro.




### Apache NetBeans IDE

[Apache NetBeans IDE](https://netbeans.apache.org/) es un entorno integrado de desarrollo y una plataforma extensible. Ofrece un soporte para varios lenguajes de programación.


Existen varias maneras de instalar Apache NetBeans IDE:

1. Utilizando los instaladores oficiales.

2. Construyéndolo a partir del código fuente.

3. Utilizando la imagen de Snap para Ubuntu.


Puede encontrar información sobre NetBeans en Snap en su sitio oficial [https://snapcraft.io/netbeans](https://snapcraft.io/netbeans).

Para instalar NetBeans desde los repositorios Snap en Ubuntu, utilice el siguiente comando:

```shell

sudo snap install netbeans --edge --classic

```

## Payara Starter

Payara Starter es un generador personalizable que permite generar proyectos JakartaEE para Payara Server o Payara Micro. 

Para generar un proyecto utilizando Payara Starter, siga los pasos a continuación:

1. Visie el sitio [https://start.payara.fish/](https://start.payara.fish/).

En la sección **Project Description**, configure las propiedades como se muestra a continuación:

   * Build:       Maven

   * Group ID:    fish.payara
   
   * Artifact ID: capitulo01

   * Version:     0.1-SNAPSHOT 

      
![](figura_01_00.png)

Presione el botón Next.

En la sección **Jakarta EE**, configure las propiedades como se muestra a continuación:

![](figura_01_01.png)

Presione el botón Next.


En la sección **Payara Platform**, configure las propiedades como se muestra a continuación:

     * Platform; Payara Micro
     * Payara Platform Versión: 6.2024.11


![](figura_01_02.png)

Presione el botón Next.


En la sección **Project Configuration**, configure las propiedades como se muestra a continuación:

* Package: fish.payara

* [x] Include Test

* Java Versión: Java SE 21

**Nota:**
Al momento de publicar este libro se ofrece soporte para Java 21.


![](figura_01_03.png)

Presione el botón Next.


En la secciión **Microprofile**, configure las propiedades como se muestra a continuación:


* Full MP


![](figura_01_04.png)

Presione el botón Next.


En la sección **Development Option**, configure las propiedades como se muestra a continuación:

* Docker

* Payara Platform version: (Seleccione la versión más reciente).

![](figura_01_05.png)

Presione el botón Next.


En la sección **ER Diagram Designer**, configure las propiedades como se muestra a continuación:

Pernite diseñar un modelo ER mediante un asistente visual.


![](figura_01_06.png)


De clic en el botón **Diagram Builder & Live Preview**, se visualiza el editor donde podrá diseñar sus modelos ER.

![](figura_01_06_1.png)

En nuestro ejemplo, remueva la marca de la casilla de Generate Code For JPA, como se muestra en la siguiente figura


![](figura_01_06_2.png)


En la sección **Advanced Configuration**, configure las propiedades como se muestra a continuación:


* Login: None


![](figura_01_06_3.png)

Haga clic en el botón **Generate**. Esto generará un archivo .zip llamado capitulo01.zip. A continuación, proceda a descomprimir este archivo.


![](figura_01_06_4.png)


2. Inicie Apache NetBeans IDE y abra el proyecto capitulo01. Para hacerlo desde el menú **File**, seleccione **Open Project**, busque el proyecto y selecciónelo.

 Podrá observar la estructura del proyecto, que se divide en varias áreas:

* WebPages: Contiene las páginas web, generalmente en formato html y faces (xhtml).

* Source Package: Alberga el código fuente de las clases Java.

* Test Package: Incluye las pruebas (test).

* Other resources: Guarda el archivo de configuración microprofile-config.properties.

* Dependencies: Lista las dependencias que se usarán en el proyecto.

* Dependencies Test: Enumera las dependencias para las pruebas.

* Project Files: Incluye el archivo pom.xml.


![](figura_01_07.png)


Encontramos la clase HelloWorldResourceTest , con un error generado, la editamos y debemos agregar las importaciones

```java
import fish.payara.resource.HelloWorldResource;
import fish.payara.resource.RestConfiguration;

```



Abra la consola del IDE cambie los permisos a los archivos mvn. Para hacerlo, seleccione el proyecto y luego, en el menú Tools, elija **Open in Terminal**.

![](figura_01_08.png)

Esto abrirá la consola de comandos donde podrá cambiar los permisos de ejecución de los archivos.

```shell

chmod 775 mvn* .*

```

3. Ahora, ejecute el siguiente comando desde la consola:

```shell

mvn clean verify

```

Se ejecutará la prueba HelloWorldResourceTest utilizando Arquillian.


![](figura_01_09.png)


Si no desea ejecutar los test, añada -Dmaven.test.skip=true  

```shell

 mvn clean verify -Dmaven.test.skip=true  

``


**Sugerencia:**

Se puede ejecutar el mismo comando desde el menú de NetBeans de dos maneras:
 
 1. Haciendo clic en el proyecto y luego en la estructura del proyecto hacer clic en el comando maven respectivo. 

 2. En menú contextual del proyecto Goal y escribir los comandos de maven.

La version actualizada de Payara ofrece el modo dev.

Ejecute desde la consola

```shell
mvn package payara-micro:dev

```

El modo dev, permite probar los test, crear las imagenes en Docker y lanzar la aplicación que permite hacer cambios en el codigo y estos se recargan de manera automaticca.

![](figura_01_09_1.png)


## Arquillian

La clase HelloWorldResourceTest, viene configurada con Arquillian para realizar los test.


Según la definición en su sitio web [https://arquillian.org/invasion/](https://arquillian.org/invasion/), Arquillian es una plataforma de pruebas innovadora y altamente extensible para la JVM. Esta plataforma permite a los desarrolladores crear de manera sencilla pruebas automatizadas de integración, funcionales y de aceptación para middleware Java.

Es fundamental destacar que las pruebas (test) son una parte esencial en el desarrollo de las aplicaciones.


![](figura_01_10.png)


Se realiza la prueba utilizando JUnit y Arquillian, para verificar que el valor devuelvo al invocar /resources/hello sea John.

```java

@Test
public void testHelloEndpoint() {
    String baseUrl = deploymentUrl.toString();

    Client client = ClientBuilder.newClient();
    Response response = client.target(baseUrl + "resources/hello")
            .queryParam("name", "John")
            .request(MediaType.TEXT_PLAIN)
            .get();

    assertEquals(Response.Status.OK.getStatusCode(), response.getStatus());
    String responseBody = response.readEntity(String.class);
    assertEquals("John", responseBody);

    client.close();
}


```

La clase que realiza la prueba necesita que el servidor, en nuestro ejemplo Payara Micro, esté en ejecución. Esta función la realiza Arquillian. Luego, realiza peticiones mediante Jakarta Web Services al endpoint resources/hello. Al pasar el valor de 'John' al parámetro 'name', debería regresar el valor de 'John'. Si no devuelve este valor, indica que el endpoint no está funcionando correctamente.


Una vez finalizada la fase de prueba, el servidor embebido será detenido automáticamente por Arquillian.

Abra la clase HelloWorldResource, que utiliza Jakarta Web Services, Microprofile Config, CDI, entre otras especificaciones, para ver su implementació.



## Jakarta Contexts and Dependency Injection

La especificación Jakarta Contexts and Dependency Injection (CDI), disponible en [https://jakarta.ee/specifications/cdi/4.0/jakarta-cdi-spec-4.0](https://jakarta.ee/specifications/cdi/4.0/jakarta-cdi-spec-4.0), se define como

Un potente conjunto de servicios complementarios que ayudan a mejorar la estructura del código de las aplicaciones. Sus características principales incluyen:

* Un ciclo de vida bien definido para objetos con estado vinculados a contextos de ciclo de vida, donde el conjunto de contextos es extensible.

* Un mecanismo de inyección de dependencias sofisticado y seguro, que incluye la capacidad de seleccionar dependencias en tiempo de desarrollo o despliegue, sin necesidad de una configuración detallada.

* Soporte para la modularidad de Jakarta EE y la arquitectura de componentes Jakarta EE - la estructura modular de una aplicación Jakarta EE se tiene en cuenta al resolver las dependencias entre los componentes Jakarta EE.

* Integración con el Lenguaje de Expresión Unificado de Jakarta (EL), permitiendo que cualquier objeto contextual sea utilizado directamente dentro de una página JSP o Jakarta Server Faces.

* La capacidad de decorar objetos inyectados (sólo en el entorno CDI Full)

* La capacidad de asociar interceptores a objetos a través de enlaces de interceptor seguros.


Estas características se ilustran en las secciones siguientes.

En esta sección, se utiliza CDI en combinación con la anotación @Inject para inyectar una propiedad a través de Microprofile-Config. La anotación @ConfigProperty se utiliza para buscar y cargar el valor de la propiedad desde el archivo de configuración.

```java

 @Inject
 @ConfigProperty(name = "defaultName", defaultValue = "world")
 private String defaultName;

```

El ejemplo inyecta el valor de la propiedad 'defaultName', que se ha definido previamente en el archivo 'microprofile-config.properties'.



```
# Define your configuration properties here
defaultName=Hello, World!

```
El archivo microprofile-config.properties es utilizado para establecer las configuraciones de la aplicación que serán leídas mediante la especificación Microprofile Config.  Se almacena dentro del directorio META-INF.

![](figura_01_12.png)


## Modificando las pruebas

Vamos a hacer un ajuste menor en la prueba. Para ello, abra la clase HelloWorldResourceTest.java y en el método testHelloEndpoint(), cambie:

```java
 assertEquals("John", responseBody); 
```

por
```java
 assertEquals("Johnx", responseBody). 
```

Luego, ejecute la prueba de nuevo utilizando el siguiente comando:

```shell

mvn clean verify


```

![](figura_01_13.png)

Como se observa en la figura, la prueba no fue exitosa. Se indica la clase, el método y la línea donde ocurrió el error. En este caso, se esperaba el valor 'John', pero se devolvió el valor de  'Johnx'.

```shell

  HelloWorldResourceTest.testHelloEndpoint:49 expected:<John[x]> but was:<John[]>

```

Restaure la prueba a su estado original.


```java
assertEquals("John", responseBody);
```


Ahora, proceda a limpiar el proyecto, ejecutar las pruebas e iniciar Payara Micro con la ejecución del proyecto.

```shell

mvn clean verify payara-micro:start

```

Podrá observar en la consola que se han ejecutado las pruebas y se ha iniciado la aplicación.

```shell

Payara Micro URLs:
http://avbravo-IdeaPad-Gaming-3-15ARH7.lan:8080/

'capitulo01-0.1-SNAPSHOT' REST Endpoints:
GET     /openapi/
GET     /openapi/application.wadl
GET     /resources/application.wadl
GET     /resources/hello

]]


```

Abra su navegador Web e ingrese a la dirección [http://localhost:8080](http://localhost:8080). Esto mostrará la página principal de nuestro proyecto.

![](figura_01_14.png)


En la sección RESTFull Services, haga clic en 'resources/hello'. Verá el siguiente mensaje en pantalla.

![](figura_01_15.png)

Ahora, vamos a enviar un valor al endpoint correspondiente al parámetro 'name'. Para hacerlo, escriba lo siguiente en la barra de dirección de su navegador.

```
http://localhost:8080/resources/hello?name=John

```

La respuesta obtenida será

```
Jhon

```

Como se puede observar, este proyecto incorpora las especificaciones de Jakarta EE y Eclipse Microprofile.


## NoSQL


Las bases de datos NoSQL están diseñadas para ofrecer escalabilidad horizontal, alto rendimiento, manejo eficiente de datos no estructurados y gestionar enormes volúmenes de datos.

Existen diversas categorías de bases de datos NoSQL entre ellas:  

* Bases de datos orientadas a documentos.  

* Bases de datos orientadas a columnas. 

* Bases de datos clave valor.  

* Bases de datos de grafos  

Estos cuatro grupos representan una amplia variedad de bases de datos que pueden ser utilizadas en el desarrollo de proyectos Java.

En este libro, nos centraremos en el uso de bases de datos orientadas a documentos.


## MongoDB

[MongoDB](https://www.mongodb.com/) es una base de datos NoSQL orientada a documentos y grafos. En lugar de utilizar tablas y registros, MongoDB emplea colecciones y documentos. Los documentos son estructuras compuestas de pares clave-valor.


A continuación, se muestra una tabla que compara MongoDB con el modelo de bases de datos relacional:

|MongoDB  | Bases Datos Relacionales |
|-----------       | -----------    |
|Base Datos      | Bases de datos        |
|Colección           | Tabla|
|Documento         | Filas           |

Entre las ventajas de las bases de datos MongoDB se encuentran:  

* Orientación a documentos.

* Su naturaleza libre de esquema.

* Su capacidad para soportar la utilización de réplicas de la base de datos. 

* Su soporte para transacciones


Consideraciones:   


* MongoDB no ofrece soporte para valores incrementales, por lo que la implementación de esta característica depende del desarrollador.

* En lugar de utilizar relaciones, MongoDB emplea un sistema de referencias entre colecciones.

* MongoDB tiene su propia sintaxis para realizar consultas.

* MongoDB almacena documentos en formato BSON, que es una representación binaria de JSON, como se muestra en el siguiente ejemplo:

```json
{
  "_id" : ObjectId("63c03a300d148c16800d60de"),
   name: "Aristides Villarreal Bravo",
   email: "avbravo@gmail.com",
   twitter: "@aristidesvbravo",
   bluesky: "avbravo.bsky.social"
   country: "Panama"
   
}

```



Este es un documento sencillo en formato BSON que representa un objeto con cinco propiedades: name, email, twitter, bluesky y country, cada una de las cuales tiene un valor de tipo String.

El campo _id es un objeto de tipo ObjectId con una longitud máxima de 12 bytes. Los primeros 4 bytes son un sello de tiempo que indica cuándo se creó el ObjectId, medido en segundos. Los 5 bytes siguientes son un valor aleatorio generado una vez por proceso único para la máquina y el proceso. Los últimos 3 bytes son un contador incremental que aumenta cada vez que se crea un nuevo ObjectId en un proceso específico.


### MongoDB Query API 

MongoDB Query API [https://www.mongodb.com/docs/manual/query-api/](https://www.mongodb.com/docs/manual/query-api/) es la herramienta que utiliza MongoDB para interactuar con los datos almacenados en sus colecciones. Esta API proporciona una variedad de métodos para consultar documentos.

Uno de estos métodos es find(), que permite realizar consultas a los documentos almacenados en una colección.


```shell

db.collection.find();

```

El método find() también puede recibir un objeto JSON que defina los criterios de búsqueda. Por ejemplo, si desea buscar a una persona llamada "Maria" en la colección "persona", utilice el siguiente código:

```shell

db.persona.find({"name":"Maria"});

```
 
### MongoDB Java Driver

MongoDB Java Driver es el controlador oficial de Java para interactuar con bases de datos MongoDB. Puede encontrar más información en el sitio [https://www.mongodb.com/docs/drivers/java-drivers/](https://www.mongodb.com/docs/drivers/java-drivers/).


Un ejemplo de cómo realizar una consulta para personas con el nombre Maria, utilizando el driver oficial de MongoDB para Java, sería el siguiente:

```java

Bson filter = Filters.eq("name", "Maria"));
persona.find(filter).forEach(doc -> System.out.println(doc.toJson()));

```



## Resumen

En este capítul es una introducción a Jakarta EE y Microprofile, Bases de datos NoSQL y se desarrolló una pequeña aplicación Jakarta EE. En el próximo capítulo, exploraremos Jmoordbcore y discutiremos los requerimientos necesarios para trabajar con este marco de trabajo.
