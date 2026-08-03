# Capítulo 11


En este capítulo, exploraremos diversas API de las especificación Microprofile. Estas APIs permiten crear métricas para monitorear el estado de los microservicios y hacer nuestras aplicaciones tolerantes a fallos, entre otras funcionalidades.


Este capítulo incluye los temas:

* Microprofile Metrics

* Fault Tolerance

* Fallback

* Health

* Ready

* Live

* Startup

* @Timed

* @Counted

* @Gauge

* @Histogram


## Microprofile Metrics


La especificación [Microprofile](https://microprofile.io/), proporciona un conjunto de APIs que podemos implementar en nuestros proyectos para trabajar con microservicios.

Le invitamos a familiarizarse con las especificaciones para adquirir un conocimiento básico de su uso. A lo largo de este capítulo, se presentan ejemplos sencillos de implementaciones.


En el capítulo 1, se hizo mención de la lista de especificaciones que ofrece Microprofile.


|Especificación                | Especificación |
|-----------                   | -----------    |
|OpenTelemetry                 | OpenAPI        |
|Rest Client                   | Config         |
|Fault Tolerance               |Metrics         |
|JWT Authentication            | Health         |
|Jakarta EE 10 Core Profile    |                |




Para consultar los endpoints, debe utilizar la sintaxis proporcionada por la especificación.

Los formatos que se admiten son: **Prometheus, OpenMetrics**.

Cabe destacar que los siguientes endpoint solo admiten solicitudes del tipo  **GET**.

|Endpoint                                       | Descripción | 
|-----------                                    |-----------  | 
|/metrics                                       |Retorna todas las métricas registradas.          |
|/metrics?scope=<scope_name>                    |Retorna todas las métricas registradas para el respectivo ámbito (base,vendor,application).           |
|/metrics?scope=<scope_name>&name=<metric_name> |Retorna las métricas que coinciden con el nombre de la métrica para el ámbito correspondiente.            |



En este capítulo, se solicita que cree un proyecto llamado **capitulo11**, siguiendo un procedimiento similar al que se utilizó para la creación de proyectos anteriores.

A continuación, cree una entidad llamada Estudiante, que contará con los siguientes atributos:

```java

@Entity()
public class Estudiante {

    @Id
    private String idestudiante;
    @Column
    private String nombre;

    @Column
    private Integer edad;
}

```

Luego, proceda a crear la interface EstudianteRepositorio, que incluirá dos métodos:

```java

@Repository(entity=Estudiante.class)
public interface EstudianteRepository extends CrudRepository<Estudiante, String>{
    @Ping
    public Boolean ping();
    @Find
    public List<Estudiante> findByNombre(String nombre);
}


```

Proceda a crear la clase EstudianteController y añada los métodos save() y findAll(). Notará que se incluyen anotaciones @Metered, las cuales forman parte de las métricas que utilizaremos. Más adelante, detallaremos su funcionalidad:

```java

@Path("estudiante")
public class EstudianteController {

  @Inject
  EstudianteRepository estudianteRepository;

  @GET
  @Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})
  @Timed(name = "estudiantesFindAll",
        description = "Monitorea el tiempo en que se obtiene la lista de todos los estudiantes",
        unit = MetricUnits.MILLISECONDS, absolute = true)
    public List<Estudiante> findAll() {

        return estudianteRepository.findAll();
    }

  @POST
  public Response save(
       @RequestBody(description = "Crea un nuevo estudiante.", content = @Content(mediaType = "application/json", schema = @Schema(implementation = Estudiante.class))) Estudiante estudiante) {
       return Response.status(Response.Status.CREATED).entity(estudianteRepository.save(estudiante)).build();
  }


}

```

A continuación, proceda a crear el UberJar del proyecto y ejecutarlo utilizando los siguientes comandos:

```shell
mvn clean payara-micro:bundle

java -jar target/capitulo11-microbundle.jar --noHazelcast


```


Posteriormente, proceda a insertar varios registros de estudiantes invocando el método save() utilizando **curl** desde consola:


```shell

curl --location --request POST 'http://localhost:8080/api/estudiante/' --header 'Content-Type: application/json' --data-raw '{"idestudiante": "1-2-3", "nombre": "Aristides", "edad": 49}'

```

```shell

curl --location --request POST 'http://localhost:8080/api/estudiante/' --header 'Content-Type: application/json' --data-raw '{"idestudiante": "7-8-5", "nombre": "Maria", "edad": 25}'


```

```shell

curl --location --request POST 'http://localhost:8080/api/estudiante/' --header 'Content-Type: application/json' --data-raw '{"idestudiante": "4-3-8", "nombre": "Luisa", "edad": 25}'

```

Para consultar todos los documentos almacenados, puede utilizar herramientas como Curl o Postman

[http://localhost:8080/api/estudiante](http://localhost:8080/api/estudiante)

![](figura_11_00.png)






## Fault Tolerance

La documentación oficial se encuentra en el siguiente enlace [Fault Tolerance](https://download.eclipse.org/microprofile/microprofile-fault-tolerance-4.0/microprofile-fault-tolerance-spec-4.0.html)

Al diseñar la arquitectura de microservicios, estos deben ser tolerables a fallos, como el agotamiento de tiempo de espera, y la necesidad de reintentar acciones. Es importante implementar comportamientos adecuados para cuando ocurren estas fallas.

La especificación proporciona una forma sencilla de implementar estas características.

La documentación oficial de la especificación destaca los siguientes aspectos:

**Tiempo de espera**: Permite definir una duración para el tiempo de espera.

**Reintento**: Permite definir un criterio sobre cuándo reintentar.

**Fallback**: Proporciona una solución para una ejecución fallida.

**CircuitBreaker**: Ofrece una forma de fallar rápidamente, interrumpiendo automáticamente la ejecución para evitar la sobrecarga del sistema y la espera indefinida o el timeout por parte de los clientes.

**Bulkhead**: Aísla los fallos en una parte del sistema, permitiendo que el resto sigue funcionando.




Vamos a crear ejemplos sencillos, iniciaremos con:

## Fallback

Esta API ofrece funcionalidades para gestionar fallos durante la ejecución de la aplicación, permitiendo especificar el método que se ejecutará en caso de fallos.

La anotación @Fallback incluye un atributo fallbackMethod, que se utiliza para indicar el método a ejecutarse en caso de fallo. Además, con la anotación @Timeout puedes especificar el tiempo durante el cual se intentará realizar la operación.



```java

import com.jmoordbcore.capitulo11.repository.EstudianteRepository;
import jakarta.enterprise.context.ApplicationScoped;
import jakarta.inject.Inject;
import jakarta.ws.rs.GET;
import jakarta.ws.rs.Path;
import org.eclipse.microprofile.faulttolerance.Fallback;
import org.eclipse.microprofile.faulttolerance.Timeout;


@Path("/faulttolerance")
@ApplicationScoped
public class FaultToleranceController {

    @Inject
    EstudianteRepository estudianteRepository;

    @Fallback(fallbackMethod = "fallback") // better use FallbackHandler
    @Timeout(500)
    @GET
    public String contadorPaises() {
        Integer count = 0;
        try {         
          count = estudianteRepository.findAll().stream().filter(p -> (p.getEdad() > 18)).map(_item -> 1).reduce(count, Integer::sum);
        } catch (Exception e) {
            //
        }
        return "Total de estudiantes mayores de 18 años: " + String.valueOf(count);
    }

    public String fallback() {

        return "Fallback respuesta por tiempo de espera agotado.";
    }
}

```


En el siguiente ejemplo, a medida que insertamos datos en la colección estudiante, los tiempos de respuesta puede aumentar. Con un volumen elevado de estudiantes, es posible que el tiempo de 500ms no sea suficiente. Cuando esto ocurre se activa el método fallback.


Realice la siguiente consulta utilizando Postman:

```

http://localhost:8080/api/faulttolerance

```

Por ejemplo, si la operación toma más tiempo del esperado, obtenemos la siguiente respuesta:

![](figura_11_01.png)

Por el contrario, si la operación se completa dentro del tiempo estipulado, se puede observar el resultado.

![](figura_11_02.png)





## Health




La especificación [Microprofile Health](https://download.eclipse.org/microprofile/microprofile-health-4.0/microprofile-health-spec-4.0.html) nos permite realizar comprobaciones sobre el estado de las implementaciones Microprofile. Generalmente, se utiliza para determinar la disponibilidad de un servicio y permite la automatización del proceso de reemplazo de un proceso por otro proceso en caso de fallos.

Los tipos de procedimiento para comprobar el estado que proporciona la especificación son:

* Comprobaciones de disponibilidad definidas con la anotación @Readiness.

* Comprobaciones de actividad definidas con la anotación @Liveness.

* Comprobaciones de inicio definidas con la anotación @Startup.


Un procedimiento HealthCheck sin ninguna de las anotaciones anteriores no es un procedimiento activo y debe ignorarse.

La inteface HealthCheck es el API principal para realizar la comprobación de la salud a nivel de aplicación:

```java

@FunctionalInterface
public interface HealthCheck {

    HealthCheckResponse call();
}

```

El tiempo de ejecución invocará el método call() a cada HealthCheck que a su vez crea un HealthCheckResponse que señala el estado de salud:

```java

public class HealthCheckResponse {

    public enum Estado { ARRIBA, ABAJO }

    public abstracto String getName();

    public abstract Estado getStatus();

    public abstract Opcional<Mapa<Cadena, Objeto>> getData();

    [...]
}

```

En las siguientes secciones, presentamos ejemplos que demuestran cómo obtener información sobre los siguientes aspectos:

* Detalles del autor.

* Espacio disponible en disco.

* Uso actual de la memoria.

* Tiempo total de tiempo de uso de la CPU por JVM.

* Estado de la conexión a la base de datos.


En el siguiente ejemplo, utilizamos la anotación @Readiness y proporcionamos información del autor a través de la clase HealthCheckResponse:

```java

@Readiness
@ApplicationScoped
public class AuthorCheck implements HealthCheck {

    @Override
    public HealthCheckResponse call() {
        return HealthCheckResponse.named("información")
                .up()
                .withData("Author", "Avbravo")
                .withData("Website", "https://avbravo.blogspot.com")
                .withData("Country", "Panamá")
                .build();
    }

}


```

En el siguiente ejemplo, se invoca al método ping() del repositorio Estudiante para verificar la conexión a la base de datos. El estado de la conexión se indica a través de los métodos up() o down() de la clase HealthCheckResponse.

```java

@Readiness
@ApplicationScoped
public class DataBaseConnectionCheck implements HealthCheck {

    @Inject
    EstudianteRepository estudianteRepository;

    @Override
    public HealthCheckResponse call() {

       if (estudianteRepository.ping()) {
          return HealthCheckResponse.up("Base de datos esta en ejecución");
       } else {
          return HealthCheckResponse.down("Base de datos esta detenida");
       }

    }
   
}

```

En el siguiente ejemplo, comprobamos el espacio disponible en el disco utilizando el método freeSpaceOfDisk() proporcionado por JmoordbCoreUtil

```java

@Readiness
@ApplicationScoped
public class DiskspaceCheck implements HealthCheck {
       
    @Override
    public HealthCheckResponse call() {
        return HealthCheckResponse.named("Espacio en disco")
                .withData("MB", JmoordbCoreUtil.freeSpaceOfDisk())
                .up()
                .build();
    }
}

```

En el siguiente ejemplo, obtenemos información acerca de la cantidad de memoria que está siendo utilizada por el proceso

```java

@Readiness
@ApplicationScoped
public class MemoryUsageCheck implements HealthCheck {
       
    @Override
    public HealthCheckResponse call() {
        return HealthCheckResponse.named("Uso de memoria")
                .withData("gb", JmoordbCoreUtil.memoryUsage())
                .up()
                .build();

    }
}


```

El siguiente ejemplo, se evalúa el tiempo de procesamiento en nanosegundos de los procesos mediante el método threadCpuTime() de la clase JmoordbCoreUtil


```java

@Readiness
@ApplicationScoped
public class ThreadCpuTimeCheck implements HealthCheck {

       
    @Override
    public HealthCheckResponse call() {
        return HealthCheckResponse.named("Tiempo de procesos JVM")
                .withData("ns", JmoordbCoreUtil.threadCpuTime())
                .up()
                .build();

    }
}

```

En el siguiente ejemplo, hacemos uso de la anotación @Liveness que devuelve un mensaje

```java

@Liveness
@ApplicationScoped
public class LiveHealthCheck implements HealthCheck {

    @Override
    public HealthCheckResponse call() {
        return HealthCheckResponse.named("live").up().build();

    }
}

```


En el siguiente ejemplo, hacemos uso de la anotación @Readiness que devuelve un mensaje

```java

@Readiness
@ApplicationScoped
public class ReadyHealthCheck implements HealthCheck {

    @Override
    public HealthCheckResponse call() {
return HealthCheckResponse.named("ready").up().build();


    }
}

```

A continuación, se presenta un ejemplo de cómo utilizar la anotación @Startup para enviar una respuesta que indica que la aplicación se ha iniciado

```java

@Startup
@ApplicationScoped
public class StartupCheck implements HealthCheck {
   @Override
   public HealthCheckResponse call() {
    return HealthCheckResponse.named("Aplicación iniciada").up().build();
   }
}



```

En la siguiente sección, vamos a consultar los resultados proporcionados por el endpoint health, utilizando la herramienta Postman

[http://localhost:8080/health](http://localhost:8080/health)


El resultado se muestra en la figura siguiente

![](figura_11_03.png)



Es posible que se observe una salida similar a la siguiente


```json
{
"status": "UP",
"checks": [
    {
        "name": "Uso de memoria",
        "status": "UP",
        "data": {
            "gb": "Initial memory: 0.12 GB Used heap memory: 0.09 GB Max heap memory: 1.89 GB Committed memory: 0.14 GB"
        }
    },
    {
        "name": "Espacio en disco",
        "status": "UP",
        "data": {
            "MB": "173618"
        }
    },
    {
        "name": "informaci�n",
        "status": "UP",
        "data": {
            "Author": "Avbravo",
            "Website": "https://avbravo.blogspot.com",
            "Country": "Panam�"
        }
    },
    {
        "name": "ready",
        "status": "UP",
        "data": {}
    },
    {
        "name": "Aplicaci�n iniciada",
        "status": "UP",
        "data": {}
    },
    {
        "name": "live",
        "status": "UP",
        "data": {}
    },
    {
        "name": "Base de datos esta en ejecuci�n",
        "status": "UP",
        "data": {}
    },
    {
        "name": "Tiempo de procesos JVM",
        "status": "UP",
        "data": {
            "ns": "87137331151"
        }
    }
]
}

```








### Ready

Ahora, ejecute consulta utilizando Postman o curl:

```
http://localhost:8080/health/ready

```

El resultado se muestra en la figura siguiente

![](figura_11_04.png)

Debería obtener una respuesta JSON similar a la siguiente

```json

{
"status": "UP",
"checks": [
    {
        "name": "Espacio en disco",
        "status": "UP",
        "data": {
            "MB": "173612"
        }
    },
    {
        "name": "Uso de memoria",
        "status": "UP",
        "data": {
            "gb": "Initial memory: 0.12 GB Used heap memory: 0.11 GB Max heap memory: 1.89 GB Committed memory: 0.14 GB"
        }
    },
    {
        "name": "Tiempo de procesos JVM",
        "status": "UP",
        "data": {
            "ns": "91159778565"
        }
    },
    {
        "name": "Base de datos esta en ejecuci�n",
        "status": "UP",
        "data": {}
    },
    {
        "name": "informaci�n",
        "status": "UP",
        "data": {
            "Author": "Avbravo",
            "Website": "https://avbravo.blogspot.com",
            "Country": "Panam�"
        }
    },
    {
        "name": "ready",
        "status": "UP",
        "data": {}
    }
]
}
```


### Live

Para comprobar el estado de vida de la aplicación, abra Postman y ejecute la siguiente consulta:

```
http://localhost:8080/health/live

```

El resultado se muestra en la siguiente figura

![](figura_11_05.png)

El resultado debería ser similar al siguiente:

```json

{
    "status": "UP",
    "checks": [
        {
            "name": "live",
            "status": "UP",
            "data": {}
        }
    ]
}


```



### Startup

Para verificar si la aplicación ha iniciado correctamente, abra Postman y ejecute la consulta:

```

http://localhost:8080/health/startup

```

La respuesta que se muestra en la siguiente figura, indica que la aplicación ha iniciado correctamente:

![](figura_11_06.png)  



Microprofile Health es útil para hacer comprobaciones sobre la salud de los microservicios, facilitando tomar decisiones oportunamente.



## Metrics



Las métricas permiten monitorizar los microservicios, facilitando la detección de problemas y la medición del comportamiento de ciertos eventos.

Las métricas deben ser configuradas en los siguientes tres grupos:

* base: Métricas descritas en la especificación que pueden proporcionar los proveedores (opcional).

* proveedor: Parámetros específicos del proveedor (opcional).

* aplicación: Parámetros específicos de la aplicación (opcional).

Este capítulo asume que usted ha leído la documentación de referencia.


Comenzaremos con ejemplos sencillos para ilustrar su uso:


## @Timed

La anotación @Timed se utiliza para medir el tiempo en que se tarda en ejecutarse un proceso. Para ver un ejemplo de su uso, consulte el método findAll() en la clase EstudianteRepository.


```java
 
@GET
@Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})
@Timed(name = "estudiantesFindAll",
        description = "Monitorea el tiempo en que se obtiene la lista de todos los estudiantes",
        unit = MetricUnits.MILLISECONDS, absolute = true)
@Operation(summary = "Obtiene todos los estudiantes", description = "Retorna todos los estudiantes disponibles")
@APIResponse(responseCode = "500", description = "Servidor inalcanzable")
@APIResponse(responseCode = "200", description = "Los estudiantes")
@Tag(name = "BETA", description = "Esta api esta en desarrollo")
@APIResponse(description = "Los estudiantes", responseCode = "200", content = @Content(mediaType = MediaType.APPLICATION_JSON, schema = @Schema(implementation = Collection.class, readOnly = true, description = "los estudiantes", required = true, name = "estudiantes")))
public List<Estudiante> findAll() {

    return estudianteRepository.findAll();
}

``` 

Pasos para medir el tiempo que consume el método findAll():

1. Inicie una consulta utilizando Curl o Postman en la siguiente dirección:

```shell

http://localhost:8080/api/estudiante

```

2. Para obtener el resultado de la métrica, ejecute una consulta en Postman utilizando el siguiente URL:


```shell

 http://localhost:8080/metrics/


```

En la figura, se muestra la información generada por todas las métricas. Deberá buscar la métrica que corresponde al nombre que definió en el parámetro 'name' de la anotación @Timed.

```java

@Timed(name = "estudiantesFindAll"

```

![](figura_11_07.png)

Los resultados de Metrics se pueden integrar fácilmente con otras herramientas de monitoreo, las cuales se exploraranen en los siguientes capítulos.



### scope=vendor

Para obtener información sobre el proveedor, ejecute la siguiente consulta mediante Curl o Postman:

```shell

http://localhost:8080/metrics?scope=vendor

```
La respuesta puede ser similar a la mostrada en la siguiente figura:

![](figura_11_08.png)


### scope=application

Para verificar las métricas específicas de la aplicación, utilice la siguiente consulta:

```

http://localhost:8080/metrics?scope=application


```

Mostrará los resultados a nivel de applicación

![](figura_11_09.png)




Si desea consultar el tiempo que tomó la ejecución del método findAll(), utilice el siguiente URL:

```
http://localhost:8080/metrics?scope=application&name=estudiantesFindAll

```

Esto generará un resultado que indica el tiempo en que demoró en ejecutar la consulta a la base de datos para obtener todos los documentos. Tenga en cuenta que el tiempo está definido en milisegundos.

![](figura_11_10.png)



## @Counted

La anotación @Counted permite contar la cantidad de veces que se invoca un método específico. Puede aplicarse a los siguientes niveles:  

* CONSTRUCTOR

* METHOD

* TYPE

Para implementar esto, edite la clase EstudianteController e incluya el siguiente método:


```java

@Path("findbynombre")
@GET
@Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})
@Counted(unit = MetricUnits.NONE,
        name = "findByNombre",
        absolute = true,
        displayName = "obtiene la cantidad de veces que se ejecuto",
        description = "Monitorea cuantas veces el método es invocado")
public List<Estudiante> findByNombre(@QueryParam("nombre") String nombre ) {
    return estudianteRepository.findByNombre(nombre);


}


```

Para comprobar la funcionalidad, realice varias consultas utilizando, por ejemplo, los siguientes comandos:


```shell

curl --location --request GET http://localhost:8080/api/estudiante/findbynombre?nombre=Ana

curl --location --request GET http://localhost:8080/api/estudiante/findbynombre?nombre=Maria

curl --location --request GET http://localhost:8080/api/estudiante/findbynombre?nombre=Luisa


```


Posteriormente, consulte todas las métricas utilizando:


```
http://localhost:8080/metrics?scope=application


```

La siguiente figura muestra en su parte inferior el recuento de las veces que se ha invocado el método:


![](figura_11_11.png)







## @Gauge

La anotación @Gauge denota un indicador, que muestra el valor del objeto anotado. Esta anotación se aplica exclusivamente a métodos.

Modifique la clase EstudianteController y agregué el siguiente código:

```java

    @Inject
    @Metric(name = "counter")
    private Counter counter;

```


Posteriormente, creé un método llamado findByIdestudiante(). Dentro de este método inserte la sentencia counter.inc(), para conocer la cantidad de veces que se ha invocado el método:


```java
 
@GET
@Path("{idestudiante}")
@Operation(summary = "Busca un estudiante por el idestudiante", description = "Busqueda de estudiante por idestudiante")
@APIResponse(responseCode = "200", description = "El estudiante")
@APIResponse(responseCode = "404", description = "Cuando no existe el idestudiante")
@APIResponse(responseCode = "500", description = "Servidor inalcanzable")
@Tag(name = "BETA", description = "Esta api esta en desarrollo")
@APIResponse(description = "El estudiante", content = @Content(mediaType = MediaType.APPLICATION_JSON, schema = @Schema(implementation = Estudiante.class)))
public Estudiante findByIdestudiante(
        @Parameter(description = "El idestudiante", required = true, example = "1", schema = @Schema(type = SchemaType.STRING)) @PathParam("idestudiante") String idestudiante) {

    counter.inc();

    return estudianteRepository.findByPk(idestudiante).orElseThrow(
            () -> new WebApplicationException("No hay estudiante con idestudiante " + idestudiante, Response.Status.NOT_FOUND));

}


```


Añada un nuevo método que utilice la anotación @Gauge. Asegúrese de definir la propiedad 'name' con el valor "estudianteCountFindByIdestudiante"


```java

@Gauge(name = "estudianteCountFindByIdestudiante", absolute = true, unit = MetricUnits.NONE)
private long count() {
    return counter.getCount();
}

```


Genere un archivo Uberjar de su proyecto utilizando el comando 

```shell

mvn clean verify payara-micro:bundle


```

Una vez generado el archivo Uberjar proceda a ejecutarlo utilizando el comando

```shell

java -jar target/capitulo11-microbundle.jar --noHazelcast

```

Realice varias consultas a su API utilizando Curl. Por ejemplo puede utilizar los siguientes comandos para obtener información de los estudiantes con identificados con '1-2-3' y '7-8-5' respectivamente.


```
curl --location --request GET http://localhost:8080/api/estudiante/1-2-3

curl --location --request GET http://localhost:8080/api/estudiante/7-8-5


```

Realice una consulta a su API utilizando Postman. Ingrese la siguiente URL:

```
http://localhost:8080/metrics?scope=application

```

![](figura_11_12.png)

La respuesta mostrará la cantidad de veces que se ha ejecutado el método específico

```shell

# TYPE estudianteCountFindByIdestudiante gauge
# HELP estudianteCountFindByIdestudiante 
estudianteCountFindByIdestudiante{mp_scope="application"} 2

```
Para realizar la consulta, puedes utilizar Curl de manera similar a como se muestra en el siguiente ejemplo:


```shell

curl -X GET http://localhost:8080/metrics?scope=application

```

Sí desea consultar únicamente la métrica correspondiente a @Gauge(name = "estudianteCountFindByIdestudiante").

Utilice el siguiente URL **/metrics?scope=<scope_name>&name=<metric_name>**.

Por ejemplo, puede utilizar Postman o Curl para ejecutar la siguiente consulta:

```shell

http://localhost:8080/metrics?scope=application&name=estudianteCountFindByIdestudiante


```
La siguiente figura muestra el resultado

![](figura_11_13.png)


## @Histogram

La anotación @Histogram facilita el cálculo de la distribución de un valor específico.

En el siguiente ejemplo, vamos a hacer cálculos de la distribución de las edades de los estudiantes utilizando paginación. Para ello, modifique la clase EstudianteRepository y defina el método findByEdadGreaterThanPagination. Este método realizará búsquedas basadas en edades mayores que un valor indicado, utilizando la clase Pagination para gestionar el desplazamiento entre páginas


```java

@Find
  public List<Estudiante> findByEdadGreaterThanPagination(Integer edad, Pagination pagination);
    
```


En la clase EstudianteController inyecte Histogram y MetricRegistry como se muestra a continuación:


```java

@Inject
@Metric(name = "edades", description = "Histograma de edades",
        displayName = "Histograma de edades de estudiantes")
private Histogram histogram;


@Inject
private MetricRegistry registry;

```

Considere el endpoint llamado 'histogram'. En este endpoint, realizaremos una consulta a la colección 'estudiante' para obtener los documentos cuya edad sea mayor que 20. Nos situamos en la primera página y especificamos que trabajaremos con 20 documentos por página.

Utilizaremos el resultado de la consulta para actualizar el histograma.


```java

@Path("histogram")
@GET
@Produces(MediaType.APPLICATION_JSON)
public Histogram histogramaEstudiante() {
try {
    var edad = 20;
    var pagina = 1;
    var registrosPorPagina = 25;
    Pagination pagination = new Pagination(pagina, registrosPorPagina);

    List<Estudiante> estudianteStream = estudianteRepository.findByEdadGreaterThanPagination(edad, pagination);
    estudianteStream.forEach(p -> {
        histogram.update(p.getEdad());
    });
} catch (Exception e) {
    ConsoleUtil.error("error " + e.getLocalizedMessage());
}

    return histogram;
}


```



Genere un archivo Uberjar de su proyecto utilizando el comando 

```shell

mvn clean verify payara-micro:bundle


```

Una vez generado el archivo Uberjar proceda a ejecutarlo utilizando el comando

```shell

java -jar target/capitulo11-microbundle.jar --noHazelcast

```


Para consultar los resultados, invocamos el endpoint 'histogram' de la siguiente manera:

```
http://localhost:8080/api/estudiante/histogram

```

![](figura_11_14.png)

Esto genera una salida que muestra el total de documentos procesados, la suma de estos, el valor máximo y la media de las edades de los estudiantes que cumplen con la condición especificada

```json

{
    "count": 3,
    "snapshot": {
        "max": 49.0,
        "mean": 33.0
    },
    "sum": 99
}

```


## Resumen

En este capítulo, aprendió a utilizar Microprofile Metrics para crear métricas y supervisar microservicios. En el próximo capítulo, exploraremos el uso de Prometheus y Grafana. Estas herramientas permiten recopilar datos y visualizarlos de manera gráfica, facilitando el monitoreo del estado de los microservicios.