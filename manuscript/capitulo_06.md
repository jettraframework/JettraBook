# Capítulo 6


Este capítulo profundiza en la gestión de fechas utilizando Jakarta RESTful Web Services, y su integración con Jmoordbcore. Se discutirán temas como el almacenamiento de fechas en el formato ISODATE en MongoDB, así como el envío de fechas como parámetros a endpoints.

Este capítulo incluye los temas:

* Jakarta RESTful Web Services con Fechas

* Anotaciones 

* Anotaciones no admitidas

* Búsquedas por fechas

* @ExcludeTime

* @IncludeTime

* Mayor que

* Mayor o igual 

* Menor

* Menor o igual
 
* Consultas con varios parámetros
 
* Mezclar @IncludeTime y @ExcludeTime
 
* Ejemplo con tres parámetros


## Jakarta RESTful Web Services con Fechas


Jmoordb admite los tipos de datos:

* java.util.Date 

* java.time.LocalDateTime.


De manera predeterminada, JmoordbCore ignora las horas, minutos y segundos al realizar búsquedas de fechas. Esto se aplica a los tipos de datos Date y LocalDateTime.

Si desea que las horas, minutos y segundos sean considerados en las búsquedas de fechas, utilice la anotación @IncludeTimes en los parámetros de los métodos.

Para indicar explícitamente que las horas, minutos y segundos no se deben tener en cuenta en las búsquedas de fechas, utilice la anotación @ExcludeTime en los parámetros de los métodos.


Definición de la interface ExcludeTime:


```java

@Documented
@Target({ FIELD, PARAMETER })
@Retention(RetentionPolicy.RUNTIME)
public @interface ExcludeTime {
    
}

```


Definición de la interface IncludeTime:

```java

@Documented
@Target({ FIELD, PARAMETER })
@Retention(RetentionPolicy.RUNTIME)
public @interface IncludeTime {
}

```

## Anotaciones 

En las siguientes anotaciones, puede indicar si desea que se incluyan las horas en las operaciones con fechas:

* @CountBy  

* @DeleteBy  

* @Find  


## Anotaciones no soportadas 


En las siguientes anotaciones para búsquedas por fecha, se utilizará una búsqueda exhaustiva que considera la fecha, la hora, los minutos y los segundos:

* @Query  

* @Delete  

* @Lookup  

* @Regex  

* @LikeBy  

* @CountLikeBy  

* @Count(Search... search)  

* @RegexCount  





Las siguientes secciones proporcionan una descripción detallada de las interfaces DateFormat y DateTimeFormat:

Definición de interface DateFormat:

```java

@Retention(RUNTIME)
@Target({ FIELD, PARAMETER })
public @interface DateFormat {
  public static final String DEFAULT_DATE = "dd-MM-yyyy";
  String value() default DEFAULT_DATE;
    
}

```


Definición de interface DateTimeFormat

```java

@Retention(RUNTIME)
@Target({ FIELD, PARAMETER })
public @interface DateTimeFormat {
    public static final String DEFAULT_DATE_TIME = "dd-MM-yyyyy'T'HH:mm:ss.SSSZZZ";
 
  String value() default DEFAULT_DATE_TIME;
}

```



En los métodos de los controladores que manejan parámetros de tipo fecha, se debe utilizar la anotación **@DateTimeFormat** para especificar el formato de fecha esperado.

En el siguiente ejemplo, se muestra cómo se puede formatear el atributo de fecha utilizando el formato 'dd-MM-yyyy':

```java

public List<Pais> findByFecha(@QueryParam("fecha") @DateFormat("dd-MM-yyyy") final Date fecha) {
          
      return paisRepository.findByFecha(fecha);
}

```



Ejemplo de cómo implementar filtros de fecha en un repositorio:

```java

@Repository(entity = Pais.class)
public interface PaisRepository extends CrudRepository<Pais, Long>{     
    
  @Query(where = "fecha .eq. @fecha")
  public List<Pais> queryByFecha(@IncludeTime Date fecha);

  @Find
  public List<Pais> findByFecha(@IncludeTime Date fecha);

  @Find
  public List<Pais> findByFecha(@ExcludeTime LocalDateTime fecha);

  @Find
  public List<Pais> findByFechaGreaterThan(Date fecha);

  @Find
  public List<Pais> findByFechaGreaterThanEquals(@ExcludeTime Date fecha);

  @Find
  public List<Pais> findByFechaLessThan(@ExcludeTime Date fecha);

  @Find
  public List<Pais> findByFechaLessThanEquals(@ExcludeTime Date fecha);

  @Find
  public Optional<Pais> findByFechaAndPais(@ExcludeTime Date fecha, String pais);

  @Find
  public List<Pais> findByFechaLessThanAndPais(@ExcludeTime Date fecha, String pais);

  @Find
  public List<Pais> findByPaisAndFechaLessThan(@ExcludeTime Date fecha, String pais);

  @Find
  public List<Pais> findByLocal(@ExcludeTime LocalDateTime local);
  
  @CountBy
  public Long countByLocal(LocalDateTime local);

  @Find
  public List<Pais> findByFechaGreaterThanAndFechaLessThan(@IncludeTime Date start, Date end);

  @Find
  public List<Pais> findByFechaGreaterThanEqualAndFechaLessThanEqual(@IncludeTime Date start, @ExcludeTime Date end);

  @Find
  public List<Pais> findByFechaGreaterThanEqualAndFechaLessThanEqualAndPais(@ExcludeTime Date start, @ExcludeTime Date end, String pais);

  @DeleteBy
  public Long deleteByFechaGreaterThan(Date fecha);

}


```

Jmoordbcore establece que se debe utilizar la anotación @IncludeTime en aquellas anotaciones que no admiten la anotación @ExcludeTime. Es importante notar que la anotación @IncludeTime puede ser omitida, ya que los métodos que no soportan @ExcludeTime la generan automáticamente de manera predeterminada:



Ejemplo utilizando @IncludeTime:

```java

@Query(where ="fecha .eq. @fecha")
public List<Pais> queryByFecha(@IncludeTime Date fecha);


```


El uso de @ExcludeTime no tendrá efecto, ya que los métodos que no lo soportan realizan por defecto la validación considerando las horas, minutos y segundos. Por lo tanto, su uso, como se ilustra en el ejemplo a continuación, no es aplicable:

```java

@Query(where ="fecha .eq. @fecha")
public List<Pais> queryByFecha(@ExcludeTime Date fecha);


```

Continuaremos trabajando,con el proyecto descrito en el capítulo 2. 

Modifique la clase PaisController.java para incluir un método que permita filtrar documentos cuya fecha sea posterior a la establecida. Tenga en cuenta que el método save() está configurado para asignar automáticamente la fecha y hora actuales del sistema. Esto se ilustra en el siguiente fragmento de código:

```java

@Path("pais")
@Tag(name = "Información del pais", description = "Endpoint para entidad Pais")
public class PaisController {

 public Response save(
        @RequestBody(description = "Crea un nuevo pais.", content = @Content(mediaType = "application/json", schema = @Schema(implementation = Pais.class))) Pais pais) {
        pais.setFecha(new Date());
        return Response.status(Response.Status.CREATED).entity(paisRepository.save(pais)).build();
    }
}
```


Por favor, detenga la ejecución del proyecto actual y luego ingrese los siguientes comandos en la consola de comandos:

```shell

mvn clean verify payara-micro:bundle

java -jar target/capitulo02-microbundle.jar --noHazelcast

```

Utilice Postman para enviar una solicitud al endpoint mediante [http://localhost:8080/api/pais/](http://localhost:8080/api/pais/).


![](figura_06_00.png)


Las fechas en el resultado se presentan en formato ISODATE:

```json
{
  "_id" : ObjectId("64ae143e943bc81a8d11e581"),
  "idpais" : NumberLong(1),
  "pais" : "Estados Unidos de America",
  "fecha" : ISODate("2023-07-12T02:47:26.426Z")
}


{
  "_id" : ObjectId("64ae1443943bc81a8d11e582"),
  "idpais" : NumberLong(2),
  "pais" : "Panamá",
  "fecha" : ISODate("2023-07-12T02:47:31.473Z")
}

{
  "_id" : ObjectId("64ae1449943bc81a8d11e583"),
  "idpais" : NumberLong(3),
  "pais" : "Colombia",
  "fecha" : ISODate("2023-07-12T02:47:37.266Z")
}

```


## Búsquedas por fechas

Al realizar búsquedas que involucran fechas, tenga en cuenta las siguientes recomendaciones:

* Utilice @IncludeTime para especificar la hora, minutos y segundos.  

* Aplique @ExcludeTime para ignorar las horas, minutos y segundos. 



## @ExcludeTime

La anotación @ExcludeTime puede ser omitida, ya que de manera predeterminada no se toman en cuenta las horas, minutos y segundos, como se muestra en el siguiente ejemplo:

```java

@Repository(entity = Pais.class)
public interface PaisRepository extends CrudRepository<Pais, Long>{     
    
   // @Find
   // public List<Pais> findByFecha(Date fecha);
  
    @Find
    public List<Pais> findByFecha(@ExcludeTime Date fecha);
  
}


```


Modifique la clase PaisController para incluir el método siguiente:


```java

@Path("fecha")
@GET
@Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})
public List<Pais> findByFecha(@QueryParam("fecha") @DateFormat("dd-MM-yyyy") final Date fecha) {
     return paisRepository.findByFecha(fecha);
}

```

Realice la consulta utilizando Curl o Postman, asegurándose de pasar el parámetro fecha sin incluir información de horas, minutos y segundos. Por ejemplo, utilice el siguiente comando:


```shell

curl http://localhost:8080/api/pais/fechahora?fecha=12-07-2023


```

Se devolverán los documentos correspondientes a la fecha especificada. Tenga en cuenta que solo se consideran la fecha (año, mes y día), excluyendo las horas, minutos y segundos. Puede introducir sus propios valores, lo que podría alterar los resultados mostrados en la sección siguiente:

```json
[
 {"fecha":"2023-07-12T02:47:26.426Z[UTC]","idpais":1,"pais":"Estados Unidos de America"},
 {"fecha":"2023-07-12T02:47:31.473Z[UTC]","idpais":2,"pais":"Panamá"},
 {"fecha":"2023-07-12T02:47:37.266Z[UTC]","idpais":3,"pais":"Colombia"}
]



```


## @IncludeTime

Se recomienda modificar el método findByFecha para que incluya el tiempo, reemplazando la anotación @ExcludeTime por @IncludeTime:

```java

//@Find
//public List<Pais> findByFecha(@ExcludeTime Date fecha);  


@Find
public List<Pais> findByFecha(@IncludeTime Date fecha);  

```

Detenga la ejecución actual del proyecto y luego reinícielo.


Por favor, vuelva a ejecutar la consulta utilizando el comando curl:


```shell
curl http://localhost:8080/api/pais/fechahora?fecha=12-07-2023

```

La consulta no devolverá ningún resultado, ya que se necesita proporcionar detalles específicos de tiempo, incluyendo horas, minutos y segundos:

```
[]

```

Por favor, edite el repositorio y vuelva a la configuración anterior utilizando la anotación @ExcludeTime


```java

@Find
public List<Pais> findByFecha(@ExcludeTime Date fecha); 

```


## Mayor que

Para este ejercicio, cambie la fecha del país 'Estados Unidos' a la fecha proporcionada:

```json

{
  "_id" : ObjectId("64ae143e943bc81a8d11e581"),
  "idpais" : NumberLong(1),
  "pais" : "Estados Unidos de America",
  "fecha" : ISODate("2023-09-15T02:47:26.426Z")
}

```


A continuación vamos a implementar un método en el repositorio PaisRepository que consulte documentos cuya fecha sea posterior a la fecha proporcionada por el parámetro fecha:

```java

@Repository(entity = Pais.class)
public interface PaisRepository extends CrudRepository<Pais, Long>{     
   @Find
   public List<Pais> findByFechaGreaterThan(Date fecha);
  
}

```


Modifique el controlador PaisController para incluir un nuevo método llamado findByFechaGreaterThan. Este método acepta un parámetro que representa una fecha, con el formato dd-MM-YYYY, que se indica mediante la anotación @DateFormat:

```java

  @Path("fechagreaterthan")
  @GET
  public List<Pais> findByFechaGreaterThan(@QueryParam("fecha") @DateFormat("dd-MM-yyyy") final Date fecha) {
    return paisRepository.findByFechaGreaterThan(fecha);
  }

```

Por favor, introduzca diferentes fechas. Realice la consulta a través de curl, Postman u otra herramienta compatible, utilice la ruta api/pais/fechagreaterthan?fecha=dd-MM-yyyy. 

```shell

curl http://localhost:8080/api/pais/fechagreaterthan?fecha=01-08-2023

```

Se devolverán dos documentos que hayan sido emitidos después del 01 de agosto de 2023:

```json
[
 {
  "fecha":"2023-09-15T02:47:26.426Z[UTC]","idpais":1,"pais":"Estados Unidos de America"
 }
]

```



## Mayor o igual 


Añada el método 'findByFechaGreaterThanEquals' en el controlador 'PaisController'. Este método se utiliza para buscar documentos con fechas iguales o posteriores a la especificada.

```java

@Path("fechagreaterthanequals")
@GET
@Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})
public List<Pais> findByFechaGreaterThanEquals(@QueryParam("fecha") @DateFormat("dd-MM-yyyy") final Date fecha) {
   return paisRepository.findByFechaGreaterThanEquals(fecha);
}

```


Ejecute la solicitud con el siguiente valor para el atributo fecha:

```shell

curl http://localhost:8080/api/pais/fechagreaterthanequals?fecha=12-07-2023

```

El resultado incluirá todos los países cuyas fechas sean iguales o posteriores a la fecha especificada:

```json
[
 {
 "fecha":"2023-09-15T02:47:26.426Z[UTC]","idpais":1,"pais":"Estados Unidos de America"
 },
 {
 "fecha":"2023-07-12T02:47:31.473Z[UTC]","idpais":2,"pais":"Panamá"
 },
 {
 "fecha":"2023-07-12T02:47:37.266Z[UTC]","idpais":3,"pais":"Colombia"
 }
]

```


## Menor

Defina el método findByFechaLessThan en el repositorio para buscar registros con fechas anteriores a la especificada.

Tenga en cuenta que @ExcludeTime es el comportamiento predeterminado para @Find, por lo que no es necesario incluirlo explícitamente.


```java

@Find
public List<Pais> findByFechaLessThan(@ExcludeTime Date fecha);

```


Ahora, en el controlador, implementé el método findByFechaLessThan:

```java

@Path("fechalessthan")
@GET
@Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})
public List<Pais> findByFechaLessThan(@QueryParam("fecha") @DateFormat("dd-MM-yyyy") final Date fecha) {        
    return paisRepository.findByFechaLessThan(fecha);
}

```


Ejecute la solicitud con el siguienta valor para el atributo fecha:

```shell

curl http://localhost:8080/api/pais/fechalessthan?fecha=13-07-2023

```


El resultado incluirá todos los países cuyas fechas sean menores a la fecha especificada:

```json
[
 {
  "fecha":"2023-07-12T02:47:31.473Z[UTC]","idpais":2,"pais":"Panamá"
 },
 {
   "fecha":"2023-07-12T02:47:37.266Z[UTC]","idpais":3,"pais":"Colombia"
  }
]

```


##  Menor o igual


Defina el método findByFechaLessThanEquals en el repositorio, para buscar registros con fechas anteriores o iguales a la especificada. Tenga en cuenta que la anotación @ExcludeTime es opcional:

```java

@Find
public List<Pais> findByFechaLessThanEquals(@ExcludeTime Date fecha);

```

Ahora, en el controlador, implementé el método findByFechaLessThanEquals:

```java

@Path("fechalessthanequals")
@GET
@Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})
public List<Pais> findByFechaLessThanEquals(@QueryParam("fecha") @DateFormat("dd-MM-yyyy") final Date fecha) {     
    return paisRepository.findByFechaLessThanEquals(fecha);
}

```

Ejecute la solicitud con el siguiente valor para el atributo fecha:


```shell

curl http://localhost:8080/api/pais/fechalessthanequals?fecha=23-09-2023

```

El resultado incluirá todos los países cuyas fechas sean menores o iguales a la fecha especificada:

```json
[
 {
  "fecha":"2023-09-15T02:47:26.426Z[UTC]","idpais":1,"pais":"Estados Unidos de America"
 }
,{
  "fecha":"2023-07-12T02:47:31.473Z[UTC]","idpais":2,"pais":"Panamá"
 },
 {
  "fecha":"2023-07-12T02:47:37.266Z[UTC]","idpais":3,"pais":"Colombia"
 }
]

```



## Consultas con varios parámetros

En la próxima sección, se proporcionan ejemplos de búsquedas que combinan fechas y otros atributos de las entidades. Por favor, modifique la clase PaisRepositorio e incluya los siguientes métodos:
 
```java
  
@Find
public Optional<Pais> findByFechaAndPais(@ExcludeTime Date fecha, String pais);

@Find    
public List<Pais> findByFechaLessThanAndPais(@ExcludeTime Date fecha, String pais);

```



Modifique el controlador PaisController para incorporar los métodos necesarios:

```java

@Path("fechaandpais")
@GET
@Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})
public Optional<Pais> findByFechaAndPais(@QueryParam("fecha") @DateFormat("dd-MM-yyyy") final Date fecha, @QueryParam("pais") String pais) {
     return paisRepository.findByFechaAndPais(fecha, pais);
}

@Path("fechalessthanandpais")
@GET
@Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})
public List<Pais> findByFechaLessThanAndPais(@QueryParam("fecha") @DateFormat("dd-MM-yyyy") final Date fecha, @QueryParam("pais") String pais) {  
       return paisRepository.findByFechaLessThanAndPais(fecha, pais);
}

```



Realice la consulta utilizando Postman:


```
http://localhost:8080/api/pais/fechaandpais?fecha=12-07-2023&pais=Colombia

```

El resultado obtenido debe ser similar a la siguiente figura:

![](figura_06_01.png)




Realice la consulta utilizando Postman:

[http://localhost:8080/api/pais/fechalessthanandpais?fecha=15-07-2023&pais=Colombia](http://localhost:8080/api/pais/fechalessthanandpais?fecha=15-07-2023&pais=Colombia)


El resultado obtenido debe ser similar a la siguiente figura:

![](figura_06_02.png)






## Mezclar @IncludeTime y @ExcludeTime

En el ejemplo siguiente, realizaremos una consulta combinando la anotación @IncludeTime para la fecha de inicio, en la que especificaremos una hora determinada. En cuanto a la fecha final, no incluiremos la hora. Tenga en cuenta que, por defecto, se utiliza la anotación @ExcludeTime.

Defina ahora en el repositorio el método 'findByFechaGreaterThanEqualsAndFechaLessThanEquals':

```java

@Find
public List<Pais> findByFechaGreaterThanEqualsAndFechaLessThanEquals(@IncludeTime LocalDateTime start, Date end);

```

A continuación, agregue el método findByFechaGreaterThanAndFechaLessThanWithoutHours() en el controlador PaisController:

```java

@Path("fechagreaterthanandfechalessthanwithouthours")
@GET
@Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})    
public List<Pais> findByFechaGreaterThanAndFechaLessThanWithoutHours(@QueryParam("fecha") @DateFormat final Date fecha,@QueryParam("fechafinal") @DateFormat final Date fechafinal) {
      LocalDateTime  start =JmoordbCoreDateUtil.dateToLocalDateTimeFirstHourOfDay(fecha);
        return paisRepository.findByFechaGreaterThanEqualsAndFechaLessThanEquals(start, fechafinal);

}

```

Realice la consulta utilizando Postman:


[http://localhost:8080/api/pais/fechagreaterthanandfechalessthanwithouthours?fecha=12-07-2023&fechafinal=28-12-2023](http://localhost:8080/api/pais/fechagreaterthanandfechalessthanwithouthours?fecha=12-07-2023&fechafinal=28-12-2023)



El resultado obtenido debe ser similar al siguiente ejemplo:

```json
[
  {
   "fecha":"2023-09-15T02:47:26.426Z[UTC]","idpais":1,"pais":"Estados Unidos de America"
  }
 ,{
   "fecha":"2023-07-12T02:47:31.473Z[UTC]","idpais":2,"pais":"Panamá"
   }
  ,{
    "fecha":"2023-07-12T02:47:37.266Z[UTC]","idpais":3,"pais":"Colombia"
   }
]


```


### Ejemplo con tres parámetros

En el siguiente ejemplo, ejecutaremos una búsqueda entre dos fechas, y el atributo de país.

Defina ahora en el repositorio el método 'findByFechaGreaterThanEqualAndFechaLessThanEqualAndPais':

```java

@Find
public List<Pais> findByFechaGreaterThanEqualAndFechaLessThanEqualAndPais(@ExcludeTime Date start, @ExcludeTime Date end, String pais);


```

Modifique el controlador PaisController para incorporar los métodos necesarios:

```java

@Path("fechagreaterthanequalandfechalessthanequalandpais")
@GET
@Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})
public List<Pais> findByFechaGreaterThanEqualAndFechaLessThanEqualAndPais(@QueryParam("fecha") @DateFormat final Date fecha, @QueryParam("fechafinal") @DateFormat final Date fechafinal, @QueryParam("pais") String pais) {
   return paisRepository.findByFechaGreaterThanEqualAndFechaLessThanEqualAndPais(fecha, fechafinal, pais);
}


```

Realice la consulta utilizando Postman:


[http://localhost:8080/api/pais/fechagreaterthanequalandfechalessthanequalandpais?fecha=12-07-2023&fechafinal=13-07-2023&pais=Colombia](http://localhost:8080/api/pais/fechagreaterthanequalandfechalessthanequalandpais?fecha=12-07-2023&fechafinal=13-07-2023&pais=Colombia)




El resultado obtenido debe ser similar al siguiente ejemplo:


```json

[
 {
   "fecha":"2023-07-12T02:47:37.266Z[UTC]","idpais":3,"pais":"Colombia"
  }
]

```

## Resumen  

Este capítulo proporciona una guía detallada sobre cómo trabajar con fechas en Jakarta RESTful Web Services. En el próximo capítulo, exploraremos cómo utilizar el controlador Java para MongoDB.




