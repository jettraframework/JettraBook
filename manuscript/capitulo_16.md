# Capítulo 16

Hoy en día, las aplicaciones en la nube demandan características como un inicio rápido y un consumo reducido de recursos. Helidon, un marco de trabajo Java ligero y orientado a la nube, cumple con estas demandas.

Este capítulo incluye los temas:

* Helidon

* Maven

* Imagen Docker

* Integración de Jmoordbcore

* Surefire-report-plugin


## Helidon

Helidon, disponible en [https://helidon.io/](https://helidon.io/), es un conjunto de librerías de código abierto diseñadas para la creación de microservicios, y es compatible con las especificaciones de Microprofile.io.

En el momento de redacción de este libro, Helidon 4.0.0 incorpora 'virtual threads', una característica de alto rendimiento disponible en Java 21.

Para utilizar Helidon 4.0.0, es imprescindible tener instalado Java 21.


Siga estos pasos para su instalación:

* Inicie descargando la versión de Java 21 desde [https://adoptium.net/es/](https://adoptium.net/es/).

* Una vez descargada, descomprímala y trasládela al directorio /usr/local siguiendo las instrucciones a continuación.



```shell

sudo mv jdk-21.0.1+12 /usr/local

```

* Después de la instalación, proceda a configurar el archivo /etc/profile de la siguiente manera:

```
export JAVA_HOME=/usr/local/jdk-21.0.1+12 

```

* Para confirmar que la instalación de Java se realizó correctamente, ejecute el siguiente comando:

```shell
java -version

```

Esto debería generar una salida similar a la siguiente:

```shell
openjdk version "21.0.1" 2023-10-17 LTS
OpenJDK Runtime Environment Temurin-21.0.1+12 (build 21.0.1+12-LTS)
OpenJDK 64-Bit Server VM Temurin-21.0.1+12 (build 21.0.1+12-LTS, mixed mode, sharing)

```




## Helidon Starter

Visite el sitio [https://helidon.io/starter/](https://helidon.io/starter/) para crear su proyecto.

![](figura_16_00.png)


En la sección correspondiente, elija la versión a utilizar **Version: 4.0.0**

**Helidon Flavor** 

* Select a Flavor: **Helidon MP**. Es la versión compatible con Microprofile

* Haga clic en el botón Next (Siguiente).


**Application Type**

* En Application Type: seleccione **Quickstart** (Contiene configuración básica). 

* Presione el botón Next (Siguiente)




**Media Support**

* En la sección Media Support : **JSON-B**. Select a JSON library: **JSON-B**. 

* Presione el botón Next (Siguiente).

**Customize Project**

* En la pestaña Customize Project,  indique los datos:

```

Project groupId: com.jmoordbcore

Project artifactId: capitulo16

Project version: 1.0-SNAPSHOT

Java package name: com.jmoordbcore.capitulo16


```

Haga clic en el botón **Download** para descargar el archivo .zip, luego proceda a descomprimir archivo.


## Maven

Además, Maven ofrece la posibilidad de crear el proyecto utilizando sus arquetipos, lo que facilita y agiliza el proceso de configuración inicial

```shell

mvn -U archetype:generate -DinteractiveMode=false \
    -DarchetypeGroupId=io.helidon.archetypes \
    -DarchetypeArtifactId=helidon-quickstart-mp \
    -DarchetypeVersion=4.0.0 \
    -DgroupId=com.jmoordbcore \
    -DartifactId=capitulo16 \
    -Dpackage=com.jmoordbcore.capitulo16

```



Abra el proyecto en su IDE preferido y ejecute el siguiente comando:

```shell

mvn package

```

Espere un momento mientras Maven inicia la descarga de las dependencias  necesarias. Cuando finalize la descarga, ingrese el siguiente comando en su consola:

```shell

java -jar target/capitulo16.jar 

```

![](figura_16_01.png)


En pocos milisegundos la aplicación es ejecutada.

Desde su navegador Web o mediante Postman, puede interactuar con el microservicio [http://localhost:8080/greet](http://localhost:8080/greet). Como se muestra en la figura siguiente

![](figura_16_02.png)

Puede consultar Microprofile Metrics, utilice [http://localhost:8080/metrics](http://localhost:8080/metrics).

Si desea consultar Microprofile Health, dirigase a la dirección [http://localhost:8080/health](http://localhost:8080/health).




## Imagen Docker
 
Para generar la imagen Docker de nuestro proyecto, ejecute el siguiente comando en la terminal.

```
docker build -t capitulo16 .

```

Para ejecutar la imagen Docker, introduzca el siguiente comando en la terminal
```
docker run --rm -p 8080:8080 capitulo16:latest

```

Para detener la ejecución de su imagen Docker, ejecute el siguiente comando

```
CRTL + c

```


## Integración de Jmoordbcore

La integración es un proceso sencillo. Como primer paso, añada la configuración de conexión a MongoDB en el archivo microprofile-config.properties

```
#---------------------------------------
# MongoDB con seguridad user y password
#mongodb.uri=mongodb://user:password@localhost:27017
#---------------------------------------
mongodb.uri=mongodb://localhost:27017
#-- Database de configuración
mongodb.jmoordb= configurationjmoordbdb
#-- Database
mongodb.database=ejemplodb
mongodb.database1=testdb


```

Abra el archivo pom.xml e incorpore el repositorio.

```xml

<repositories>
    <repository>
        <id>jitpack.io</id>
        <url>https://jitpack.io</url>
    </repository>
</repositories>

```


Incluya la propiedad y la dependencia de jmoordbcore en su archivo de configuración

```xml

<version.jmoordbcore>2.0.2</version.jmoordbcore>

<dependency>
    <groupId>com.github.avbravo</groupId>
    <artifactId>jmoordb-core</artifactId>
    <version>${version.jmoordbcore}</version>
</dependency>

```

Posteriormente, construya la entidad Estudiante siguiendo el mismo procedimiento que se utilizó en el capítulo 11.

```java

@Entity()
public class Estudiante {

    @Id
    private String idestudiante;
    @Column
    private String nombre;

    @Column
    private Integer edad;
    
    
//set/get

}

```


Tenga en cuenta que es necesario crear la clase de repositorio 'AutogeneratedRepository' para gestionar los valores secuenciales.

```java
@AutosecuenceRepository(entity = Autosequence.class, jakartaSource = JakartaSource.JAKARTA)
public interface AutogeneratedRepository {

    @Autogenerated()
    public Long generate(String database, String collection);

}

```

Proceda a definir la interfaz 'EstudianteRepository


```java

@Repository(entity=Estudiante.class)
public interface EstudianteRepository extends CrudRepository<Estudiante, String>{


    @Find
    public List<Estudiante> findByNombre(String nombre);
    @Find
    public List<Estudiante> findByEdadGreaterThanPagination(Integer edad, Pagination pagination);
}


```



Implemente la clase **MongoDBProducer** para gestionar la conexión con MongoDB.

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

Continúe con la creación del Controlador para definir los Endpoints.

```java

@Path("estudiante")
public class EstudianteController {

    @Inject
    EstudianteRepository estudianteRepository;


    @GET
    @Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})

    public List<Estudiante> findAll() {
        return estudianteRepository.findAll();
    }

    @GET
    @Path("{idestudiante}")
    public Estudiante findByIdestudiante(@PathParam("idestudiante") String idestudiante) {
        return estudianteRepository.findByPk(idestudiante).orElseThrow(
            () -> new WebApplicationException("No hay estudiante con idestudiante " + idestudiante, Response.Status.NOT_FOUND));

    }

    @POST
    public Response save(@RequestBody(description = "Crea un nuevo estudiante.", content = @Content(mediaType = "application/json", schema = @Schema(implementation = Estudiante.class))) Estudiante estudiante) {
        return Response.status(Response.Status.CREATED).entity(estudianteRepository.save(estudiante)).build();
    }

    @PUT
    public Response update(
      @RequestBody(description = "Crea un nuevo estudiante.", content = @Content(mediaType = "application/json", schema = @Schema(implementation = Estudiante.class))) Estudiante estudiante) {

        return Response.status(Response.Status.CREATED).entity(estudianteRepository.update(estudiante)).build();
    }

    @DELETE
    @Path("{idestudiante}")
    public Response delete(@PathParam("idestudiante") String idestudiante) {
        estudianteRepository.deleteByPk(idestudiante);
        return Response.status(Response.Status.NO_CONTENT).build();
    }

    @GET
    @Path("findbynombre")
    @Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})
    public List<Estudiante> findByNombre(@QueryParam("nombre") String nombre) {
       return estudianteRepository.findByNombre(nombre);

    }

}



```

Compile y ponga en marcha el proyecto utilizando los comandos que se indican a continuación.

```shell

mvn clean verify

java -jar target/capitulo16.jar 

```

Ejecute la consulta en [http://localhost:8080/estudiante](http://localhost:8080/estudiante) utilizando Postman o Curl. Obtendrá un resultado similar al que se muestra en la figura siguiente

![](figura_16_03.png)


Proceda a realizar una consulta por nombre de estudiante utilizando una URL similar a

```
http://localhost:8080/estudiante/findbynombre?nombre=Maria
 
```

Genera la respuesta siguiente:

```json
[
    {
        "edad": 25,
        "idestudiante": "7-8-5",
        "nombre": "Maria"
    }
]

```

La creación de aplicaciones con Helidon es un proceso relativamente sencillo que resulta en un alto rendimiento.


## Consideraciones sobre Lookup

Anteriormente, se explicó el uso de lookup, que permite procesar consultas de tipo MongoDB Query Language directamente pasándolos como parámetros. Continuación edite EstudianteRepository y agregue el método lookup

```java

import com.jmoordb.core.annotation.repository.Find;
import com.jmoordb.core.annotation.repository.Lookup;
import com.jmoordb.core.annotation.repository.Repository;
import com.jmoordb.core.model.Pagination;
import com.jmoordb.core.model.Search;
import com.jmoordb.core.repository.CrudRepository;
import com.jmoordbcore.capitulo16.model.Estudiante;
import java.util.List;

/**
 *
 * @author avbravo
 */
@Repository(entity = Estudiante.class)
public interface EstudianteRepository extends CrudRepository<Estudiante, String> {

   @Find
   public List<Estudiante> findByNombre(String nombre);

   @Find
   public List<Estudiante> findByEdadGreaterThanPagination(Integer edad, Pagination pagination);

   @Lookup
   public List<Estudiante> lookup(Search search);
}


```

Modifique la clase EstudianteController y agregue el método

```java

@GET
@Path("lookup")
@Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})
public List<Estudiante> lookup(@QueryParam("filter") String filter, @QueryParam("sort") String sort, @QueryParam("page") Integer page, @QueryParam("size") Integer size) {
  List<Estudiante> suggestions = new ArrayList<>();
  try {
      Search search = DocumentUtil.convertForLookup(filter, sort, page, size);
      suggestions = estudianteRepository.lookup(search);
  } catch (Exception e) {
      MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + "error: " + e.getLocalizedMessage());
  }
  return suggestions;
}

```

Construiremos una consulta

```
http://localhost;8080/estudiante/lookup?filter={"nombrer": {"$eq": "Maria"}}&sort={"nombre": -1}&page=1&size=1
```

A diferencia de Payara-Micro, Helidon no acepta los caracteres {} directamente por lo cual tendremos que realizar su conversión correspondiente, nos guiaremos de la siguiente tabla

| Caracter | Representación|
| -------- | ------- |
| %7B      | {    |
| %22      | :    |
| %7D      | }    |


El resultado final sería reemplazando las llaves { y } por los caracteres.


```
http://localhost:8080/estudiante/lookup?filter=%7B"nombre":%7B"$eq": "Maria"%7D%7D&sort=%7B"nombre": -1%7D&page=1&size=1

```

La siguiente figura muestra su uso desde Postman

![](figura_16_07.png)


## Test

Helidon ofrece una API que facilita la realización de pruebas en las aplicaciones.

Verifique en el archivo pom.xml para asegurarse de que tiene la dependencia necesaria.


```

 <dependency>
    <groupId>io.helidon.microprofile.tests</groupId>
    <artifactId>helidon-microprofile-tests-junit5</artifactId>
    <scope>test</scope>
</dependency>

```

La anotación @HelidonTest inicia automáticamente el servidor para ejecutar las pruebas. Una vez que las pruebas se han ejecutado, el servidor se detiene. 

@WebTarget nos facilita la realización de invocaciones al servidor y la gestión de sus respuestas.

Vamos a verificar que, al proporcionar un valor como **Maria** al Endpoint /estudiante/findbynombre, se nos devuelve un documento que coincide con el mostrado en el paso anterior.

A continuación, se presenta el código de la clase que hemos desarrollado para ejecutar esta prueba

```java

@HelidonTest
class EstudianteTest {

    @Inject
    private MetricRegistry registry;

    @Inject
    private WebTarget target;

    @Test
    void findByNombre() {

        String result = target
                .path("estudiante/findbynombre")
                .queryParam("nombre", "Maria")
                .request()
                .get(String.class);
        assertThat(result, is("[{\"edad\":25,\"idestudiante\":\"7-8-5\",\"nombre\":\"Maria\"}]"));

    }

}


```

Ejecutamos las pruebas desde la consola utilizando el siguiente comando

```shell

mvn clean verify

```

Si las pruebas son exitosas, se generará el archivo target/capitulo16.jar. Sin embargo, si las pruebas fallan, este archivo no se producirá.

```shell

[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.277 s - in com.jmoordbcore.capitulo16.EstudianteTest
[INFO] 
[INFO] Results:
[INFO] 
[INFO] Tests run: 6, Failures: 0, Errors: 0, Skipped: 0
[INFO] 
[INFO] 
[INFO] --- dependency:3.6.0:copy-dependencies (copy-libs) @ capitulo16 ---

```

Continúe modificando la prueba, reemplazando el nombre 'Maria' por 'Ana'. Dado que este nombre no existe, se espera que la prueba falle

```java

 @Test
void findByNombre() {

    String result = target
            .path("estudiante/findbynombre")
            .queryParam("nombre", "Ana")
            .request()
            .get(String.class);
    assertThat(result, is("[{\"edad\":25,\"idestudiante\":\"7-8-5\",\"nombre\":\"Maria\"}]"));

}


```

Puede llevar a cabo la ejecución de la prueba directamente desde la consola.

```shell

mvn clean verify

```
Se producirá un mensaje de error que indica que la prueba no ha sido superada, tal y como se ilustra a continuación.

```shell

[INFO] Results:
[INFO] 
[ERROR] Failures: 
Tests run: 1, Failures: 1, Errors: 0, Skipped: 0, Time elapsed: 0.312 s <<< FAILURE! - in com.jmoordbcore.capitulo16.EstudianteTest
com.jmoordbcore.capitulo16.EstudianteTest.findByNombre  Time elapsed: 0.132 s  <<< FAILURE!
java.lang.AssertionError: 

Expected: is "[{\"edad\":25,\"idestudiante\":\"7-8-5\",\"nombre\":\"Maria\"}]"
     but: was "[]"
[INFO] 
[ERROR] Tests run: 6, Failures: 1, Errors: 0, Skipped: 0
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------


```

Si está utilizando NetBeans IDE, seleccione el proyecto y luego haga clic en 'Test'. Esto ejecutará el proyecto y mostrará visualmente el resultado de la prueba.

![](figura_16_04.png)


Modifique de nuevo la prueba para que utilice 'Maria'.

```java

 @Test
void findByNombre() {



String result = target
        .path("estudiante/findbynombre")
        .queryParam("nombre", "Maria")
        .request()
        .get(String.class);
assertThat(result, is("[{\"edad\":25,\"idestudiante\":\"7-8-5\",\"nombre\":\"Maria\"}]"));

}
```


## Surefire-report-plugin
Surefire-report-plugin es un plugin Maven para la generación de informes de las pruebas, los cuales pueden ser integrados con otras herramientas. 

El procedimiento es bastante sencillo: simplemente añada los siguientes plugins al archivo pom.xml.

```xml
<reporting>
   <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-surefire-report-plugin</artifactId>
        <version>3.1.2</version>
        <configuration>
           <showSuccess>true</showSuccess>
            <outputDirectory>target/surefire-reports</outputDirectory>
            <linkXRef>false</linkXRef>
        </configuration>  

        </plugin>

       <plugin>
         <groupId>org.apache.maven.plugins</groupId>
         <artifactId>maven-site-plugin</artifactId>
         <version>4.0.0-M10</version>
         <configuration>
            <outputDirectory>${basedir}/target/surefire-reports</outputDirectory>
        </configuration>
       </plugin>
    </plugins>
</reporting>

```


Desde la consola, ejecute el comando que se muestra a continuación

```shell

mvn clean site

```

Posteriormente, proceda a ejecutar el comando correspondiente

```shell

mvn surefire-report:report  

```

Podrá notar que en la carpeta 'target' se crea una subcarpeta llamada 'site'. Esta contiene el archivo 'surfire-report.html', que es el informe, y la carpeta 'sufire-reports', que incluye los informes de las pruebas en formatos xml y txt.

![](figura_16_05.png)

Abra el archivo 'surfire-report.html' en su navegador para encontrar un informe que es fácil de revisar.

![](figura_16_06.png)


## Resumen

En este capítulo, hemos explorado cómo implementar Jmoordbcore con Helidon, generar imágenes para Docker y llevar a cabo pruebas. En el siguiente capítulo, profundizaremos en la creación de microservicios utilizando OpenLiberty.


