# Capítulo 18


Este capítulo abarca el uso de las especificaciones Jakarta Data y Jakarta NoSQL con MongoDB.



Este capítulo incluye los temas:


* Jakarta Data


* Jakarta NoSQL


* Jmooordb-core-jnosql



## Jakarta Data


La especificación Jakarta Data disponible en [http://www.jnosql.org/spec/](http://www.jnosql.org/spec/), ofrece una API que facilita el acceso a las bases de datos. No es un reemplazo para Jakarta Persistence ni para Jakarta NoSQL, sino para complementar estas API.


Jakarta Data consta de tres interfaces, siendo la interfaz principal DataRepository la interfaz principal.


```
                          DataRepository<T,K>
                             |
          ___________________|______________________
         |                                          |
CrudRepository<T, K>                      ReactiveRepository<T,K>
         |
         |
PageableRepository<T,K>


```


En este capítulo hacemos uso de CrudRepository<T,K>, el cual incluye las implementaciones de las operaciones más comunes.

## Jakarta NoSQL


Jakarta NoSQL es la especificación Java diseñada para bases de datos NoSQL. Ofrece soporte para una amplia variedad de bases de datos NoSQL.


Se integra con funcionalidades con Jakarta Data.


Eclipse NoSQL disponible en [http://www.jnosql.org/docs/document.html](http://www.jnosql.org/docs/document.html), ofrece implementaciones para interactuar bases de datos NoSQL.


Simplemente, necesitarás añadir la siguiente dependencia:


```xml

<dependency>
  <groupId>org.eclipse.jnosql.databases</groupId>
  <artifactId>jnosql-mongodb</artifactId>
  <version>1.0.1</version>
</dependency>


```


Y agregar la configuración correspondiente al archivo microprofile-config.properties


```

jnosql.document.database=databases
jnosql.mongodb.host=localhost:27017


```


## Jmooordb-core-jnosql


Jmooordb-core-jnosql ofrece soporte para Jmoordbcore y se integra con Eclipse JNoSQL, permitiendo emplear ambas implementaciones en un mismo proyecto.

Al añadir la dependencia jmoordb-core-jnosql, ya se incorpora jnosql-mongodb.

```xml

<dependency>
    <groupId>com.github.avbravo</groupId>
    <artifactId>jmoordb-core-jnosql</artifactId>
    <version>1.6.0</version>
</dependency>


<repositories>
    <repository>
        <id>jitpack.io</id>
        <url>https://jitpack.io</url>
    </repository>
</repositories>


```

La próxima sección presenta un ejemplo que utiliza Payara Micro y Jmoordb-core-jnosql, aprovechando las APIs de Eclipse JNoSQL.



## Proyecto de Ejemplo


Cree un nuevo proyecto de la misma manera que se hizo en los capítulos anteriores, siguiendo estos pasos:


En NetBeans IDE, seleccione File luego seleccione New Project y especifique las siguuientes propiedades:


```

Categories: Java with Maven

Projects: Web Application

```

Haga clic en el botón Siguiente y nombra al proyecto como: **capitulo18**.


Una vez generado el proyecto, selecciónelo y de clic derecho para seleccionar el menú New, seleccione Other. 

Aparecerá un cuadro de diálogo que despliega el asistente. Indique las siguientes propiedades:


```
Categories: Payara
File Types: Payara Micro Maven Plugin

```

Haga clic en el botón Siguiente, seleccione Payara Micro Version: 6.2024.11, y luego presione el botón Finish.


Abra el archivo pom.xml e indique el nombre capitulo18 en la propiedad <name>capitulo18</name>.


En la sección <build>, añada lo siguiente


```
<finalName>capitulo18</finalName>


```


Compruebe que el proyecto funciona correctamente ejecutándolo desde la consola

```shell

mvn clean verify payara-micro:bundle

java -jar target/capitulo18-microbundle.jar 

```


Posteriormente, proceda a detener la ejecución del proyecto.


A continuación, añade la dependencia:


```xml

<dependency>
    <groupId>com.github.avbravo</groupId>
    <artifactId>jmoordb-core-jnosql</artifactId>
    <version>1.6.0</version>
</dependency>

```

No olvide agregar la configuración necesaria al archivo microprofile-config.properties, ya que Eclipse JNoSQL requiere que se establezcan las siguientes propiedades:


```xml

jnosql.document.database=ejemplodb
jnosql.mongodb.host=localhost:27017

```

Para utilizar Jmoordbcore, necesita agregar:


```xml
#---------------------------------------
# MongoDB con seguridad user y password
#mongodb.uri=mongodb://user:password@localhost:27017
#---------------------------------------
mongodb.uri=mongodb://localhost:27017
#-- Database de configuración
mongodb.jmoordb= configurationjmoordbdb
#-- Database
mongodb.database=ejemplodb

```

Ahora, proceda a crear la entidad Estudiante. Se utilizará Eclipse NoSQL, por lo que los imports serán diferentes a los de Jmoordbcore.


```java

@Entity(value = "estudiante")
public class Estudiante {

    @Id("idestudiante")
    private String idestudiante;
    @Column
    private String nombre;

    @Column
    private Integer edad;

    public Estudiante() {

    }
    //set/get
          
}

```


Para el repositorio, emplearemos Jakarta Data, como se muestra en el siguiente segmento de código.


```java

import com.jmoordb.capitulo18.model.Estudiante;
import jakarta.data.repository.CrudRepository;
import jakarta.data.repository.OrderBy;
import jakarta.data.repository.Repository;
import java.util.List;

@Repository
public interface EstudianteRepository extends CrudRepository<Estudiante, String>{

List<Estudiante> findByNombre(String nombre);

  @OrderBy("edad")
  List<Estudiante> findByNombreLike(String nombrePattern);
  
}

```

Defina la clase EstudianteController donde se establecerán los endpoints para la colección estudiante.


```java

@Path("estudiante")
@Tag(name = "Información del estudiante", description = "Endpoint para entidad Estudiante")
public class EstudianteController {

    @Inject
    EstudianteRepository estudianteRepository;

    @GET
    @Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})

    public List<Estudiante> findAll() {
        return estudianteRepository.findAll().toList();
    }


    @GET
    @Path("{idestudiante}")
    public Estudiante findByIdestudiante(
         @Parameter(description = "El idestudiante", required = true, example = "1", schema = @Schema(type = SchemaType.STRING)) @PathParam("idestudiante") String idestudiante) {


       return estudianteRepository.findById(idestudiante).orElseThrow(

             () -> new WebApplicationException("No hay estudiante con idestudiante " + idestudiante, Response.Status.NOT_FOUND));


    }


    @POST
    public Response save(

        @RequestBody(description = "Crea un nuevo estudiante.", content = @Content(mediaType = "application/json", schema = @Schema(implementation = Estudiante.class))) Estudiante estudiante) {



      return Response.status(Response.Status.CREATED).entity(estudianteRepository.save(estudiante)).build();
    }


    @PUT
    public Response update(

      @RequestBody(description = "Crea un nuevo estudiante.", content = @Content(mediaType = "application/json", schema = @Schema(implementation = Estudiante.class))) Estudiante estudiante) {

      return Response.status(Response.Status.CREATED).entity(estudianteRepository.save(estudiante)).build();
    }


    @DELETE
    @Path("{idestudiante}")
    public Response delete(

      @Parameter(description = "El elemento idestudiante", required = true, example = "1", schema = @Schema(type = SchemaType.STRING)) @PathParam("idestudiante") String idestudiante) {

       estudianteRepository.deleteById(idestudiante);
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


Edite la clase JakartaRestConfiguration.java y cambie  ApplicationPath a **("api")** , como se muestra en el siguiente código


```java

@ApplicationPath("api")
public class JakartaRestConfiguration extends Application {

    
}

```


Inicia el proyecto utilizando los siguientes comandos:

```shell

mvn clean verify payara-micro:bundle

java -jar target/capitulo18-microbundle.jar 

```

Consulte los endpoint utilizando Postman o Curl.


```shell

 curl --location --request GET 'http://localhost:8080/api/estudiante/' 


```


Se produce una lista de los documentos de la colección estudiante.

```json

[

{"edad":49,"idestudiante":"1-2-3","nombre":"Aristides"},
{"edad":25,"idestudiante":"7-8-5","nombre":"Maria"},
{"edad":25,"idestudiante":"4-3-8","nombre":"Luisa"}
]

```


Jakarta NoSQL, ofrece una variedad de características, entre ellas Fluent API y  record.

Como ejemplo, usaremos record para definir la entidad Estudiante de la siguiente forma:


```java

@Entity(value = "estudiante")
public record Estudiante (@Id("idestudiante")String idestudiante, @Column String nombre, @Column  Integer edad){
    
}


```


## Resumen


Este capítulo aborda el uso de la especificación Jakarta NoSQL con Eclipse NoSQL y jmoordb-core-jnosql. Se crean endpoints utilizando las especificaciones de Eclipse Microprofile. Al implementar Jakarta NoSQL en tus proyectos, obtendrá muchos beneficios.








