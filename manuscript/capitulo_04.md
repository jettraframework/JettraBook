# Capítulo 4

En este capítulo, exploraremos el patrón repositorio. Discutiremos su importancia e implementación en Jmoordbcore.

Este capítulo incluye los temas:

* ¿Qué es un Repositorio?

* @Repository

* @Save  

* @Update

* @UpdateMany

* @Count   

* @CountBy   

* @CountLikeBy   

* @SearchCountLikeBy  

* @Delete

* @DeleteBy

* @DeleteMany

* @Find  

* @LikeBy  

* @SearchLikeBy

* @Lookup  

* @Ping  

* @Query    

* @Regex  

* @RegexCount  



## ¿Qué es un Repositorio?


En el sitio Web de Martin Flower, encontrarás un artículo escrito por 'Edward Hieatt and Rob Mee' [https://martinfowler.com/eaaCatalog/repository.html](https://martinfowler.com/eaaCatalog/repository.html), donde explican que el patrón repositorio media entre el dominio y las capas de mapeo de datos, utilizando una interfaz de tipo colección para acceder a los objetos del dominio. Este patrón facilita la abstracción de la complejidad de la interacción con la base de datos, al permitir el uso de interfaces para encapsular las operaciones de acceso a datos.

En Jmoordbcore, usted define una interfaz repositorio utilizando la anotación @Repository para cada entidad y en ella declara los métodos que representan las operaciones en la base de datos.

Durante el proceso de compilación del proyecto, Jmoordbcore se encarga de generar automáticamente el código necesario para implementar los métodos que fueron previamente definidos en las interfaces del proyecto.



## @Repository

La interface esta definida mediante

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.SOURCE)
public @interface Repository {

   Class<?> entity();

    JakartaSource jakartaSource() default JakartaSource.JAKARTA;
    ConfigEngine configEngine() default ConfigEngine.MICROPROFILE_CONFIG;

    String collection() default "";
    String database() default "{mongodb.database}";

}



```




La anotación @Repository contiene los siguientes atributos: 


|Atributo          | Descripción | 
|-----------       | -------------------------------------------------------    |
|                  |                |
|entity()          |Define la clase de la entidad Entity.class    |
|jakartaSource()   |Especifique las fuentes de Jakarta o Java EE Legacy |            
|configEngine()    |Especifique el motor para trabajar con el archivo microprofile-config.properties mediante MICROPROFILE_CONFIG o JETTRA_CONFIG |            
|collection()      |Nombre de la colección. Vacío indica que es el mismo nombre del entity. |            
|database()        |Nombre de la base de datos a usar lo tomará de microprofile-config **mongodb.database** |


**Nota:**

* Es recomendable utilizar MICROPROFILE_CONFIG para servidores Jakarta EE/Eclipse Microprofile tales como Helidon, Payara, Wildfly,OpenLiberty, Quarkus, TommEE.

* Para JettraServer se recomienda establecer JETTRA_CONFIG como motor para trabajar con el archivo de configuración.







## Anotaciones 


Anotaciones para utilizar con una interface Repository son las siguientes:


```java

@Save  
@Update
@UpdateMany
@Count   
@CountBy   
@CountLikeBy   
@SearchCountLikeBy  
@Delete
@DeleteBy
@DeleteMany
@Find  
@LikeBy  
@SearchLikeBy
@Lookup  
@Ping  
@Query    
@Regex  
@RegexCount  
@Repository  


```


    


## CrudRepository

Jmoordbcore, ofrece una interfaz CrudRepository que contiene los métodos más comunes para las operaciones de inserciones, actualizaciones, eliminaciones y búsquedas.

A continuación se muestra la interfaz CrudRepository.




```java

public interface CrudRepository<T, PK> {
    public Optional<T> save(T t);

    public Boolean update(T t);

    public List<T> findAll();

    public List<T> findAllPagination(Pagination pgntn);

    public List<T> findAllSorted(Sorted sorted);

    public List<T> findAllPaginationSorted(Pagination pgntn, Sorted sorted);

    public Optional<T> findByPk(PK pk);

    public Long deleteByPk(PK pk);

    public Long deleteMany(Search search);

    public Long updateMany(Bson bson, Bson bson1);

    public JmoordbException getJmoordbException();
}


```

A continuación, se presentan ejemplos de cómo se implementan las interfaces repositorio:


* Ejemplo de una interfaz definida con un atributo primario de tipo String.


 
```java

@Repository(entity = Oceano.class, jakartaSource = JakartaSource.JAKARTA,
            database = "{mongodb.database}", collection = "oceano")
public interface OceanoRepository extends CrudRepository<Oceano, String>{  
}       

```
 


* Ejemplo de un repositorio definido con un atributo primario de tipo Long. En este caso, se pueden omitir los atributos JakartaSource, database y collection.

   
```java       

@Repository(entity = Pais.class)
public interface PaisRepository extends CrudRepository<Pais, Long>{
    
}

```
 
### Como usar un Repositorio

CDI de Jakarta EE permite utilizar inyección de dependencias. Esto permite inyectar un repositorio mediante la anotación @Inject.

En el siguiente ejemplo se inyecta el repositorio OceanoRepository y se invoca al método save() para guardarlo en la colección de MongoDB.
 
```java

@Inject
OceanoRepository oceanoRepository;

public String save(Oceano oceano){
    oceanoRepository.save(oceano);
}

```



## Especificar base de datos

Para especificar la base de datos a utilizar en el repositorio, configure el atributo **mongodb.database** en el archivo microprofile-config.properties.

```java
@Repository(entity = Persona.class, jakartaSource =  JakartaSource.JAKARTA, database ="{mongodb.database}",  collection = "persona")
public interface PersonaRepository {

    @Query(where = "")
    public List<Persona> findAll();

}
```

Si necesita utilizar otra base de datos, defina un nuevo atributo **mongodb.database1**, y especifíquelo en el atributo database de la anotación @Repository, como se muestra a continuación:

 
```java
@Repository(entity = Persona.class,  database ="{mongodb.database1}", collection = "persona")
public interface PersonaRepository {

    @Query(where = "")
    public List<Persona> findAll();
}
```
 
De esta forma, puede especificar múltiples bases de datos, por ejemplo **mongodb.database2**.
 
```java
@Repository(entity = Persona.class, database ="{mongodb.database2}")
public interface PersonaRepository {

    @Query(where = "")
    public List<Persona> findAll();
}
```
 

En la próxima sección, se detalla el uso de las anotaciones que pueden ser usadas con una interfaz definida con la anotación @Repository.

## @Save

Un método que usa la anotación @Save, tiene como función crear un nuevo documento.

Retorna un valor de tipo Boolean u Optional, que indica si fue exitosa o no la operación.

@Save admite un parámetro que es de tipo de la entidad definida.

Definición de la interface Save:
 
```java

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Save { }

```

Ejemplos de cómo utilizar el método save con valores de retorno diferentes:

* Sin utilizar CrudRepository
 
```java

@Repository(entity = Oceano.class)
public interface OceanoRepository  {
    @Save 
    public Optional<Oceano> save(Oceano oceano);

    @Save 
    public Boolean saveOceano(Oceano oceano);
}
```
 
* Al utilizar CrudRepository, no es necesario definir el método save:
 
```java

@Repository(entity = Oceano.class)
public interface OceanoRepository extends CrudRepository<Oceano,String> {
}

```
 

## @Update 

Un método que usa la anotación @Update, tiene como función actualizar un documento. 

Retorna un valor de tipo Boolean, que indica si fue exitosa o no la operación.

@Update admite un parámetro que es de tipo de la entidad definida.

Definición de la anotación:
 
```java

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Update { }

```
 
Ejemplo de uso del método update:


```java

@Repository(entity = Oceano.class)
public interface OceanoRepository  {
    @Update
    public Boolean update(Oceano oceano);
}

```
 


Al utilizar CrudRepository, no es necesario definir el método update:
 
```java

@Repository(entity = Oceano.class)
public interface OceanoRepository extends CrudRepository<Oceano,String> {
}

```
 

## @UpdateMany
 
El método UpdateMany permite la actualización de múltiples documentos. Este método se utiliza en combinación con CrudRepository, que lo genera automáticamente, eliminando la necesidad de definirlo manualmente.

En el siguiente segmento de código se muestra una sentencia para actualizar el atributo estado con base a la condición que la edad sea mayor que 18.


```java

@Repository(entity = Persona.class)
public interface PersonaRepository extends CrudRepository<Persona,String> {

}

```

En el controlador, se invoca un método específico, pasándole los atributos query y update como parámetros.


```java

Bson query = gt("edad", 18);
Bson update = Updates.combine(
               Updates.set("estado", "Mayor de edad"));

personaRepository.updateMany(query, update);
```

## @Delete

El método realizará una operación de eliminación de uno o más documentos. Los nombres de los métodos deben comenzar con **delete**. Este método devolverá un valor Long que representa el número total de documentos eliminados.

El atributo **where()** se utiliza para aplicar condiciones que se definen en una consulta @Query, excepto por (paginación y sorted). Este atributo puede recibir un parámetro de tipo Search(no se puede utilizar con otros parámetros). En este caso, el where() estará vacío.


Definición de la anotación:
 
```java

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Delete {
  String  where() default "";
}

```
 

Los operadores siguientes deben estar delimitados entre los caracteres *.* . A continuación, se presenta la lista de operadores, organizados por grupos:
|                   |                                | 
|-------------------| -----------------------------------------------    |
| **Comparación**   | **Lógicos**                                        |
|.eq.               |.and.                                               |
|.ne.               |.or.                                                |            
|.lt.               |.not.                                               |            
|.lte.              |                                                    |     
|.gt.               |                                                    |     
|.gte.              |                                                    |     



Los operadores no permitidos son: **( ) $ [ ]**.

En el siguiente ejemplo, se define una entidad con algunos atributos:

```java

@Entity
public class Oceano {
    @Id
    private String idoceano;
    @Column
    private String oceano;
    @Column
    Date fecha;
    @Column
    String activo;
    @Column
    Integer km;
    
    public Oceano() {
    }
    //set/get
}


```

A continuación, se presenta la interfaz PersonaRepository. Esta interfaz muestra varias implementaciones del método delete:

```java
@Repository(entity = Persona.class)
public interface PersonaRepository extends CrudRepository<Persona,String> {

    @Delete(where = "idoceano .eq. @idoceano")
    public Long delete(String idoceano);

    @Delete(where = "idoceano .ne. @idoceano")
    public Long deleteNEQ(String idoceano);

    @Delete(where = "idoceano .lt. @idoceano")
    public Long deleteLT(String idoceano);

    @Delete(where = "idoceano .lte. @idoceano")
    public Long deleteLTE(String idoceano);

    @Delete(where = "idoceano .gt. @idoceano")
    public Long deleteGT(String idoceano);

    @Delete(where = "idoceano .gte. @idoceano")
    public Long deleteGTE(String idoceano);

    @Delete(where ="idoceano .eq. @idoceano .and. oceano .ne. @oceano")
    public Long delete(String idoceano, String oceano);

    @Delete(where ="idoceano .eq. @idoceano .and. oceano .ne. @oceano .not. fecha .gt. @fecha")
    public Long deleteIdOceanoAndOceanoNotFecha(String idoceano, String oceano, Date fecha);

    @Delete(where = "idoceano .eq. @idoceano .and. oceano .ne. @oceano  .not. fecha .gt. @fecha .or. activo .ne. @activo")
    public Long deleteIdOceanoAndOceanoNotFechaOrActivo(String idoceano, String oceano, Date fecha, String activo);

    @Delete(where = "idoceano .eq. @idoceano .and. oceano .ne. @oceano .not. fecha .gt. @fecha .or. activo .eq. @activo .and. km .gt. @km")
    public Long deleteIdOceanoAndOceanoNotFechaOrActivoAndKm( String idoceano, String oceano, Date fecha, String activo, Integer km);

    @Delete()
    public Long delete(Search search);
}
```
 

## @CountBy


La anotación @CountBy se utiliza para contar los documentos que cumplen con una condición específica, la cual se define en el nombre del método.

Las siguientes son reglas para utilizar @CountBy:

* Los nombres de métodos deben iniciar en **countBy**. 

* Nombres de **campo** comienzan con letra mayúscula. 

* El comparador se coloca después del nombre de **campo**. Si se omite se asume que es Equal.

**Comparadores:**

* Equal : Igual.

* NotEqual: Diferente.

* GreaterThan: Mayor que.

* GreaterThanEqual: Mayor o igual.

* LessThan: Menor que.

* LessThanEqual: Menor o igual que.

* Like: Busca parte del texto.


**Operadores Lógicos:**

* And: Y

* Or: O

* Not: No


JmoordbCore analizará el nombre del método y los parámetros para verificar si la sintaxis es correcta, y generará el código correspondiente. El tipo de valor de retorno es Long.

Definición de la anotación:
 
```java

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface CountBy { }

```
 

Ejemplos de uso de CountBy:

 
```java

@Repository(entity = Oceano.class) 
public interface OceanoRepository {
  
@CountBy
public Long countByIdOceano(String idoceano);

@CountBy
public Long countByIdOceanoNotEqual(String idoceano);

@CountBy
public Long countByIdOceanoAndOceanoNotEqualAndDate(String idoceano,String oceano, Date date);

@CountBy
public Long countByIdOceanoAndOceano(String idoceano, String oceano);

@CountBy
public Long countByIdOceanoAndOceanoNotFechaGreaterThan(String idoceano, String oceano, Date fecha);

@CountBy
public Long countByIdOceanoAndOceanoNotFechaOrActivo(String idoceano, String oceano, Date fecha, String activo);

@CountBy
public Long countByIdOceanoAndOceanoNotFechaOrActivoAndKm(String idoceano, String oceano, Date fecha, String activo, Integer km);

}

```
 

## @DeleteBy

La anotación @DeleteBy se utiliza para eliminar los documentos que cumplen con una condición específica, la cual se define en el nombre del método.

Las siguientes son reglas para utilizar @DeleteBy:


* Los nombres de métodos deben iniciar en **deleteBy**.

* Los nombres de **campo** inician con letra mayúscula. 

* El comparador se coloca después del nombre de campo, (Si se omite se asume que es Equal).

  Los comparadores permitidos son: (Equal, NotEqual, GreaterThan, GreaterThanEqual, LessThan, LessThanEqual, Like).

  Los operadores Lógicos permitidos son: (And, Or, Not).


* Valores de retorno son List<>, Set<>, Optional<>, Stream<>.

JmoordbCore analizará el nombre del método y los parámetros para verificar si la sintaxis es correcta, y generará el código correspondiente.

Definición de la anotación:
 
```java

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface DeleteBy { }

``` 
 

Ejemplo de uso de @DeleteBy: 

 
```java

@Repository(entity = Oceano.class)
public interface OceanoRepository {

@DeleteBy
public Long deleteByIdOceanoAndKm(String idoceano, Integer km);

@DeleteBy
public Long deleteByIdOceanoAndOceanoNotEqualAndDate(String idoceano, String oceano, Date date);

@DeleteBy
public Long deleteByIdOceano(String idoceano);

@DeleteBy
public Long deleteByIdOceanoNotEqual(String idoceano);

@DeleteBy
public Long deleteByIdOceanoAndOceano(String idoceano, String oceano);

@DeleteBy
public Long deleteByIdOceanoAndOceanoNotFechaGreaterThan(String idoceano, String oceano, Date fecha);

@DeleteBy
public Long deleteByIdOceanoAndOceanoNotFechaOrActivo(String idoceano, String oceano, Date fecha, String activo);

@DeleteBy
public Long deleteByIdOceanoAndOceanoNotFechaOrActivoAndKm(String idoceano, String oceano, Date fecha, String activo, Integer km);

}

```
 
## @DeleteMany
 
El método DeleteMany permite la eliminación de múltiples documentos. Este método se utiliza en combinación con CrudRepository, que lo genera automáticamente, eliminando la necesidad de definirlo manualmente.


## @Query

La anotación @Query permite definir las consultas personalizadas.

Las siguientes son las reglas para utilizar @Query:

* Los métodos deben iniciar con **query**. 

* @Query devuelve una colección de tipo List< Entity > o Set < Entity >u Optional< > o Stream< >. 

* Jmoordbcore verifica que los valores del atributo where() estén escritos correctamente.

* Los objetos **Pagination** y **Sorted** no se especifican en el where(), sino que se definen como parámetros del método.
 
* Los operadores deben estar encerrados entre **.** .

Listado de operadores:

|  **Comparación**  | **Lógicos**         | **Desplazamiento**| 
|-------------------| --------------------|-------------------|
|                   |                     |                   |
|.eq.               |.and.                |   .order. .by.    |
|.ne.               |.or.                 |   .limit. .skip.  |         
|.lt.               |.not.                |                   |
|.lte.              |                     |                   |
|.gt.               |                     |                   |
|.gte.              |                     |                   |


Los operadores no permitidos son: **(  ) $ [  ]**.


Definición de la anotación:

 
```java

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Query {
  String  where() default "";
}

```




A continuación, se presenta un listado de los métodos de consulta (query) más utilizados:
 
```java

@Query()
public List<Oceano> queryAll();

@Query()
public Set<Oceano> queryAllSet();

@Query()
public Stream<Oceano> queryAllSteam();

@Query(where = "idoceano .eq. @idoceano")
public Optional<Oceano> queryByIdoceano(String idoceano);

@Query(where = "oceano .eq. @oceano ")
public List<Oceano> queryByOceano(String oceano);

@Query(where = "oceano .eq. @oceano ")
public Set<Oceano> queryByOceanoSet(String oceano);

@Query(where = "idoceano .eq. @idoceano .and. oceano .eq. @oceano")
public List<Oceano> queryByIdoceanoAndOceano(String idoceano,String oceano);

@Query(where = "idoceano .eq. @idoceano .and. oceano .ne. @oceano .not. fecha .gt. @fecha")
public List<Oceano> queryByIdOceanoAndOceanoNotFecha(String idoceano, String oceano, Date fecha);

@Query(where = "idoceano .eq. @idoceano .and. oceano .ne. @oceano .not. fecha .gt. @fecha .or. activo .ne. @activo")
public List<Oceano> queryByIdOceanoAndOceanoNotFechaOrActivo(String idoceano, String oceano, Date fecha, String activo);

@Query(where = "idoceano .eq. @idoceano .and. oceano .ne. @oceano .not. fecha  .gt. @fecha .or. activo .eq. @activo  .and. km .gt. @km")
public List<Oceano> queryByIdOceanoAndOceanoNotFechaOrActivoAndKm(String idoceano, String oceano, Date fecha,String activo, Integer km);
 
@Query()
public List<Oceano> queryAllPagination(Pagination pagination);

@Query(where = "idoceano .eq. @idoceano")
public List<Oceano> queryByIdOceanoPagination(String idoceano, Pagination pagination);

@Query(where = "idoceano .eq. @idoceano .and. oceano .eq. @idoceano")
public List<Oceano> queryByIdoceanoAndOceanoPagination(String idoceano, String oceano, Pagination pagination);

@Query(where = "idoceano .eq. @idoceano .and. oceano .ne. @oceano .not. fecha .gt. @fecha .or. activo .eq. @activo  .and. km .gt. @km")
public List<Oceano> queryByIdOceanoAndOceanoNotFechaOrActivoAndKmPagination(String idoceano, String oceano, Date fecha, String activo, Integer km, Pagination pagination);

@Query()
public List<Oceano> queryAllOrder(Sorted sorted);

@Query(where = "idoceano .eq. @idoceano")
public List<Oceano> queryByIdOceanoSorted(String idoceano, Sorted sorted);

@Query(where = "idoceano .eq. @idoceano .and. oceano .eq. @oceano .not. fecha .gt. @fecha .or. activo .eq. @activo .and. km .gt. @km")
public List<Oceano> queryByIdOceanoAndOceanoNotFechaOrActivoAndKmSorted( String idoceano, String oceano, Date fecha, String activo, Integer km, Sorted sorted);

@Query()
public List<Oceano> queryAllPaginationSorted(Pagination pagination,Sorted sorted);

@Query(where = "oceano .eq. @oceano")
public List<Oceano> queryByOceanoPagination(String oceano,Pagination pagination, Sorted sorted);

@Query(where = "idoceano .eq. @idoceano .and. oceano .eq. @oceano")
public List<Oceano> queryByIdOceanoPaginationSorted(String idoceano,String oceano, Pagination pagination, Sorted sorted);

```
 
**Nota**:  

Utilice una sola línea para definir las anotaciones con sus atributos. En el libro, se muestran en varias líneas para ajustar el espaciado y tamaño de las páginas.

Para realizar consultas de fechas y consultas complejas, se recomienda usar @Lookup en lugar de @Query. 

Jmoordbcore verifica mientras usted escribe los métodos que estos cumplan las reglas de sintaxis.

En la siguiente figura, observará que el nombre de uno de los parámetros no es adecuado. Esto genera una advertencia.


![](figura_04_03.png)



## @Find 

La anotación @Find se utiliza para consultar los documentos que cumplen con una condición específica, la cual se define en el nombre del método.

Las siguientes son reglas para utilizar @Find:

* Los nombres de métodos deben iniciar por **findAll** o **findBy**. 

* Los nombres de **campo** inician en letra mayúscula.

* El comparador se coloca después del nombre de campo. Si se omite se asume que es Equal.

  Comparadores: (Equal, NotEqual, GreaterThan, GreaterThanEqual, LessThan, LessThanEqual,Like).

  Operadores Lógicos: (And, Or, Not).

* JmoordbCore analizará el nombre del método y los parámetros para verificar la sintaxis. Posteriormente, generará el código correspondiente.

* @Find devuelve una colección de tipo List< Entity > o Set < Entity >u Optional< > o Stream< >. 

Definición de la anotación:

 
```java

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Find { }

```
 

El siguiente ejemplo muestra diversos métodos con la anotación @Find:

 
```java

@Repository(entity = Oceano.class)
public interface OceanoRepository {

@Find()
public List<Oceano> findAll();

@Find()
public Set<Oceano> findAll();

@Find()
public Stream<Oceano> findAll();

@Find()
public List<Oceano> findAllPagination(Pagination pagination);

@Find()
public List<Oceano> findAllSorted(Sorted sorted);

@Find()
public List<Oceano> findAllPaginationSorted(Pagination pagination,Sorted sorted);

@Find()
public Optional<Oceano> findByIdoceanoNotEqual(String idoceano);

@Find()
public Optional<Oceano> findByIdoceano(String idoceano);

@Find()
public Optional<Oceano> findByIdoceanoNotEqual(String idoceano);

@Find()
public List<Oceano> findByOceano(String oceano);

@Find()
public Set<Oceano> findByOceano(String oceano);

@Find()
public List<Oceano> findByIdoceanoAndOceano(String idoceano, String oceano);

@Find()
public List<Oceano> findByIdOceanoAndOceanoNotFecha(String idoceano, String oceano, Date fecha);

@Find()
public List<Oceano> findByIdOceanoAndOceanoNotFechaOrKmLessthan(String idoceano, String oceano, Date fecha, Integer km);

@Find()
public List<Oceano> findByIdOceanoAndOceanoNotFechaGreaterThanAndFechaLessThanOrKmLessthan(String idoceano,String oceano, Date fecha,Date fecha2, Integer km);

@Find()
public List<Oceano> findByIdOceanoAndOceanoAndFechaGreaterThan(String idoceano,String oceano, Date fecha);

@Find()
public List<Oceano> findByIdOceanoAndOceanoNotFechaOrActivo(String idoceano, String oceano, Date fecha,String activo);

@Find()
public List<Oceano> findByIdOceanoAndOceanoNotFechaOrActivoAndKm(String idoceano, String oceano, Date fecha,String activo,Integer km);

@Find()
public List<Oceano> findByIdOceanoPagination(String idoceano, Pagination pagination);

@Find()
public Set<Oceano> findByOceanoPagination(String idoceano,Pagination pagination);

@Find()
public List<Oceano> findByIdOceanoNotEqualPagination(String idoceano,Pagination pagination);

@Find()
public List<Oceano> findByIdOceanoSorted(String idoceano, Sorted sorted);

@Find()
public List<Oceano> findByIdOceanoPaginationSorted(String idoceano,Pagination pagination, Sorted sorted);

@Find()
public List<Oceano> findByIdoceanoAndOceanoPagination(String idoceano, String oceano, Pagination pagination);

@Find()
public List<Oceano> findByIdOceanoAndOceanoNotFechaOrActivoAndKmPagination( String idoceano, String oceano,Date fecha,  String activo,  Integer km, Pagination pagination);

@Find()
public List<Oceano> findByIdOceanoAndOceanoNotFechaOrActivoAndKmPaginationSorted(String idoceano,String oceano,Date fecha, String activo, Integer km,Pagination pagination, Sorted sorted);

@Find()
public List<Oceano> findByIdOceanoAndOceanoNotFechaOrActivoAndKmOrIdiomaNotEqualPaginationSorted(String idoceano,String oceano,Date fecha, String activo, Integer km,String idioma, Pagination pagination,Sorted sorted);

@Find()
public List<Oceano> findByIdOceanoAndOceanoNotFechaOrActivoAndKmSorted(String idoceano, String oceano,  Date fecha, String activo, Integer km,Sorted sorted);

```
 


## @Lookup

La anotación @Lookup facilita construir consultas complejas de tipo MongoDB BSON utilizando Filter de MongoDB [https://www.mongodb.com/docs/manual/reference/operator/aggregation/filter/](https://www.mongodb.com/docs/manual/reference/operator/aggregation/filter/).

Esta anotaciçon acepta un parámetro de tipo Search que incluye filtros, paginación y ordenación.

Una de sus principales ventajas es que permite la interacción con JAX-RS, lo que facilita el envío de consultas complejas en formato JSON.


Los resultados de estas consultas pueden ser devueltos en List< Entity >, Set< Entity > y Stream< >.

Definición de la anotación:
 
```java

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Lookup{
}

```
 
Definición de la clase Search.java:


```java 

public class Search {
    Document filter;
    Pagination pagination = new Pagination();
    Sorted sorted = new Sorted();
    public Search() {
    }
   // set/get    
}

```
   
Ejemplo de un repositorio que implementa @Lookup:

 
```java

@Lookup
public List<Oceano> lookup(Search search);

@Lookup
public Set<Oceano> lookupSet(Search search);

@Lookup
public Stream<Oceano> lookupStream(Search search);

```


En el siguiente ejemplo se muestra como crear un filtro con ordenación mediante el uso de Lookup, para se convierte a objeto a tipo Search los operandos y realiza la consulta. 

```java

Bson filtert = new Document("oceano", "pacifico").append("active", Boolean.TRUE);
Document sort = new Document("idoceano", 1);

Search search = DocumentUtil.convertForLookup(filter sort, 0, 0);

var result = oceanoRepository.lookup(search);

```

Ahora, anticipamos brevemente la explicación de cómo usar lookup en microservicios y en un cliente en los próximos capítulos. 

El método DocumentUtil.convertForLookup tiene la función de convertir los parámetros recibidos en formato JSON en una instancia de la clase Search, por ejemplo:

```java

@GET
@Path("lookup")
@Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})
public List<Appconfiguration> lookup(@QueryParam("filter") String filter, @QueryParam("sort") String sort,
   @QueryParam("page") Integer page, @QueryParam("size") Integer size) {
      List<Appconfiguration> suggestions = new ArrayList<>();
      try {
        Search search = DocumentUtil.convertForLookup(filter, sort, page, size);
        suggestions = appconfigurationRepository.lookup(search);
        } catch (Exception e) {
         MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + "error: " + e.getLocalizedMessage());
        }  
        return suggestions;
    }

```
El capítulo 15 profundiza en el uso del Microprofile RestClient. En el siguiente ejemplo se demuestra cómo pasar parámetros al endpoint para su posterior conversión a objetos de tipo Search.

```java

@RegisterRestClient()
@Path("/appconfiguration")
@ClientHeaderParam(name = "Authorization", value = "{lookupAuth}")
public interface AppconfigurationClient {

 @GET
 @Path("lookup")
 public List<Appconfiguration> lookup(@QueryParam("filter") String filter, @QueryParam("sort") String sort,@QueryParam("page") Integer page, @QueryParam("size") Integer size);
            
}

```


En el ejemplo a continuación, se presenta la clase LoginFaces que incorpora AppConfigurationClient. Se puede observar que el método searchAppConfiguration() emplea la función encodeBson() para codificar los datos, que luego se envían al endpoint.

```java
@Named
@SessionScoped
@Data
public class LoginFaces implements Serializable {

@Inject
UserClient userClient;

@Inject
AppconfigurationClient appconfigurationClient;


public Boolean searchAppConfiguration() {
  Boolean result = Boolean.FALSE;
  try {

     Integer page = 1;
     Integer size = 1;
     Bson filter = new Document("applicative.idapplicative", idapplicative.get().longValue());
     Document sort = new Document("applicative.idapplicative", 1);
     List<Appconfiguration> list = appconfigurationClient.lookup(                    
               EncodeUtil.encodeBson(filter),
               EncodeUtil.encodeBson(sort),
               page, size);


    if (list == null || list.isEmpty()) {
         new Appconfiguration();
    } else {
         appconfigurationLogged = list.get(0);
        result = Boolean.TRUE;
    }
   } catch (Exception e) {
     FacesUtil.errorMessage(FacesUtil.nameOfMethod() + " " + e.getLocalizedMessage());
     System.out.println(FacesUtil.nameOfMethod() + " " + e.getLocalizedMessage());
  }
   return result;
 }

```


## @Regex

La anotación @Regex, se emplea para realizar búsquedas basadas en expresiones regulares usando el comando $regex en MongoDB. [https://www.mongodb.com/docs/manual/reference/operator/query/regex/](https://www.mongodb.com/docs/manual/reference/operator/query/regex/).

Es usada comúnmente para operaciones de tipo autocompletado en componentes Web.

El valor de retorno es List< Entity >  o Set< Entity >  o Stream< > .

Definición de la anotación:
 
```java

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Regex {
    String where();
    CaseSensitive caseSensitive() default CaseSensitive.NO;
    TypeOrder typeOrder() default TypeOrder.ASC;
    LikeByType likeByType() default LikeByType.FROMTHESTART;
}

```

Observe el siguiente segmento de código:

```java
@Regex(where = "oceano .like. @oceano ",caseSensitive = CaseSensitive.NO,typeOrder = TypeOrder.ASC)
public List<Oceano> regex(String oceano);

``` 

 Mediante where() se crean filtros para las consultas. Su sintaxis es: **atributo_del_documento  .like. @parametro**.  El tipo de valor del primer atributo debe ser un String que representa al campo que se va a implementar la expresión regular.

 CaseSensitive: Se utiliza para indicar si se tomara o no en cuenta mayúsculas o minúsculas en la consulta:

  CaseSensitive.NO: No se tomará en cuenta mayúscula o minúsculas.

  CaseSensitive.YES: Se tomará en cuenta mayúscula o minúsculas.

* TypeOrder: Se usa para indicar el orden ascendente o descendente de los resultados.

  ASC:  Ordena ascendentemente con base en el campo indicado.
 
  DESC: Ordena descendentemente con base en el campo indicado.

* Puede contener un segundo parámetro que es de tipo Pagination.  
  
```java
atributo_del_documento .like. @parametro .limit. pagination .skip. @pagination  

```

* Con LikeByType puede personalizar la búsqueda mediante:

```
FROMTHESTART: Inicia la búsqueda desde el primer carácter del texto.

FROMTHEEND: Realiza con la búsqueda en el final del texto,

ANYWHERE: Busca en cualquier lugar del texto.

```

Nota: 

 La anotación @Regex se asemeja a la instrucción LIKE de SQL, por ejemplo:
 
```sql

select * from oceano where oceano LIKE "pac%"

select * from oceano where oceano LIKE "pac%" limit 1 skip 5

```
   

A continuación se muestra ejemplo del uso de @Regex:

```java

@Regex(where = "oceano .like. @oceano  ",caseSensitive = CaseSensitive.NO,typeOrder = TypeOrder.ASC)
public List<Oceano> regex(String oceano);

@Regex(where = "oceano .like. @oceano  ",caseSensitive = CaseSensitive.NO,typeOrder = TypeOrder.ASC)
public Stream<Oceano> regexStream(String oceano);

@Regex(where = "oceano .like. @oceano  ", caseSensitive = CaseSensitive.YES,typeOrder = TypeOrder.DESC)
public List<Oceano> regexSensitiveOrder(String oceano);

@Regex(where = "oceano .like. @oceano ", caseSensitive = CaseSensitive.YES,typeOrder = TypeOrder.DESC)
public Set<Oceano> regexOceano(String oceano);

@Regex(where = "oceano .like. @oceano ", caseSensitive = CaseSensitive.NO,typeOrder = TypeOrder.ASC)
public List<Oceano> regexPagintation(String oceano, Pagination pagination);

@Regex(where = "oceano .like. @oceano", caseSensitive = CaseSensitive.YES,typeOrder = TypeOrder.DESC)
public List<Oceano> regexPagintationSorted(String oceano,Pagination pagination);

@Regex(where = "oceano .like. @oceano  ", caseSensitive = CaseSensitive.YES,typeOrder = TypeOrder.DESC)
public List<Oceano> regexSensitiveOrder(String oceano);


@Regex(where = "oceano .like. @oceano ",caseSensitive = CaseSensitive.YES,typeOrder = TypeOrder.DESC)
public Set<Oceano> regexOceano(String oceano);

@Regex(where = "oceano .like. @oceano .limit. pagination .skip. @pagination",caseSensitive = CaseSensitive.NO, typeOrder = TypeOrder.ASC)
public List<Oceano> regexPagintation(String oceano, Pagination pagination);

@Regex(where = "oceano .like. @oceano .limit. pagination .skip. @pagination",caseSensitive = CaseSensitive.YES, typeOrder = TypeOrder.DESC)
public List<Oceano> regexPagintationSorted(String oceano,Pagination pagination);

```  
  


## @LikeBy

LikeBy permite definir las consultas utilizando el análisis del nombre de métodos.

Se utiliza para búsquedas basadas en expresiones regulares usando el comando [$regex en MongoDB](https://www.mongodb.com/docs/manual/reference/operator/query/regex/).  

Es usada comúnmente operaciones de autocompletado tipo autocomplete, para componentes Web.

El valor de retorno es List< Entity >  o Set< Entity >  o Stream< > .


Definición de la anotación:

 
```java

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface LikeBy {
    CaseSensitive caseSensitive() default CaseSensitive.NO;
    TypeOrder typeOrder() default TypeOrder.ASC;
    LikeByType likeByType() default LikeByType.FROMTHESTART;
}

```

* CaseSensitive: Se utiliza para indicar si se tomara o no en cuenta mayúsculas o minúsculas en la consulta:

  CaseSensitive.NO: No se tomará en cuenta mayúscula o minúsculas.

  CaseSensitive.YES: Se tomará en cuenta mayúscula o minúsculas.

* TypeOrder: Se usa para indicar el orden ascendente o descendente.

  ASC:  Ordena ascendentemente con base en el campo indicado.

  DESC: Ordena descendentemente con base en el campo indicado.

* Puede contener un segundo parámetro que es de tipo Pagination.  
  
A continuación se muestra ejemplo del uso de @LikeBy:
 
```java

@LikeBy(caseSensitive = CaseSensitive.NO, typeOrder = TypeOrder.ASC)
public List<Oceano> likeByOceano(String oceano);

@LikeBy(caseSensitive = CaseSensitive.NO, typeOrder = TypeOrder.ASC, LikeByType.FROMTHEEND)
public Stream<Oceano> likeByIdOceano(String idoceano);


@LikeBy(caseSensitive = CaseSensitive.YES, typeOrder = TypeOrder.ASC)
public Set<Oceano> likeByOceanoPagination(String oceano, Pagination pagination);


```
  



## @SearchLikeBy

La anotación @SearchLikeBy permite definir las consultas basadas en el análisis del nombre de métodos, incorporando filtros, ordenación y paginación mediante un objeto de tipo Search. 

búsquedas están basadas en expresiones regulares usando el comando [$regex en MongoDB](https://www.mongodb.com/docs/manual/reference/operator/query/regex/)  

Se utiliza principalmente en operaciones de tipo autocomplete con múltiples filtros y con paginación.

Los resultados de la búsqueda son List< Entity >  o Set< Entity >  o Stream< > .

La ordenación especificada en el objeto Search tiene prioridad sobre cualquier otra ordenación definida. Si no se establece ninguna ordenación en el objeto Search, se utilizará la ordenación definida en la anotación. 

En SQL, se traduciría a una consulta como la siguiente:

 
```sql

select * from oceano where oceano LIKE "pac%" and campo2 = valor order by campo1 limit 1 skip 3

```



Definición de la anotación:

 
```java

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface SearchLikeBy {
    CaseSensitive caseSensitive() default CaseSensitive.NO;
    TypeOrder typeOrder() default TypeOrder.ASC;
    LikeByType likeByType() default LikeByType.FROMTHESTART;
}

```
  

A continuación, se muestra un ejemplo del uso de @SearchLikeBy:
 
```java

@SearchLikeBy(caseSensitive = CaseSensitive.NO, typeOrder = TypeOrder.ASC)
public List<Oceano> likeByOceano(String oceano, Search search);

@LikeBy(caseSensitive = CaseSensitive.NO, typeOrder = TypeOrder.ASC, likeByType = LikeByType.FROMTHEEND)
public Stream<Oceano> likeByIdOceano(String idoceano, Search search);

@SearchLikeBy(caseSensitive = CaseSensitive.NO, typeOrder = TypeOrder.ASC, likeByType = LikeByType.ANYWHERE)
public List<Oceano> searchLikeByActivo(String activo, Search search);

```



En un controlador JAX-RS, implemente un endpoint que convierta los parámetros a un objeto de tipo 'Search'. Luego, invoque el método 'searchByActivo()' del repositorio. Este proceso se muestra en el siguiente ejemplo:

```java

@GET
@Path("likebyoceanopagination")
@Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})
public List<Oceano> searchLikeByActivo(@QueryParam("activo") String activo, @QueryParam("filter") String filter, @QueryParam("sort") String sort, @QueryParam("page") Integer page, @QueryParam("size") Integer size) {
    List<Ocenao> suggestions = new ArrayList<>();
    try {
        Search search = DocumentUtil.convertForLookup(filter, sort, page, size);
        suggestions = oceanboRepository.searchLikeByActivo(activo, search);
    } catch (Exception e) {
        MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + "error: " + e.getLocalizedMessage());
    }
    return suggestions;
}


```


## @CountLikeBy

La anotación @CountLikeBy, cuenta el número de documentos que coinciden con una expresión regular especificada. Se suele utilizar en combinación con la anotación @LikeBy, con operaciones de desplazamiento con paginación.

La anotación devuelve un valor de tipo Long.

Definición de la anotación:

 
```java

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface CountLikeBy {
    CaseSensitive caseSensitive() default CaseSensitive.NO;
    LikeByType likeByType() default LikeByType.FROMTHESTART;
}

```
  

Ejemplo de uso:

 
```java

@Repository(entity = Oceano.class)
public interface OceanoRepository {

   @CountLikeBy(caseSensitive = CaseSensitive.NO)
   public Long countLikeByOceano(String oceano);

   @CountLikeBy(caseSensitive = CaseSensitive.NO,LikeByType.ANYWHERE)
   public Long countLikeByIdOceano(String idoceano);

}
```
 
## @SearchCountLikeBy

La anotación @SearchCountLikeBy, cuenta el número de documentos que coinciden con una expresión regular especificada que utiliza Search, se asocia al uso de @SearchLikeBy.

La anotación devuelve un valor de tipo Long.


Definición de la anotación:

 
```java

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface SearchCountLikeBy {
   CaseSensitive caseSensitive() default CaseSensitive.NO;

   LikeByType likeByType() default LikeByType.FROMTHESTART;
}


```
  

Ejemplo de SearchCountLikeBy:

 
```java

@Repository(entity = Oceano.class)
public interface OceanoRepository {

   @SearchCountLikeBy(caseSensitive = CaseSensitive.NO)
   public Long searchCountLikeByOceano(String oceano, Search search);

   @SearchCountLikeBy(caseSensitive = CaseSensitive.NO,LikeByType.ANYWHERE)
   public Long serchCountLikeByIdOceano(String idoceano, Search search);

}

```
  

## @Count(Search... search)

Cuenta los documentos de una colección. Solo se permite un parámetro opcional de tipo Search. Si no se proporciona el parámetro Search, el método cuenta todos los documentos en la colección.

La anotación devuelve un valor de tipo Long. 

Definición de la anotación:

 
```java

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Count {
  
}

```
  

Ejemplo de uso:

 
```java
@Repository(entity = Oceano.class)
public interface OceanoRepository {
    @Count()
    public Long count(Search... search);
}

```
  

## @CountBy

La anotación @CountBy cuenta los documentos basándonos en el análisis del nombre de métodos.

Para utilizarla correctamente, debe seguir las siguientes reglas:

* Los nombres de métodos deben iniciar con **countBy**.  

* El nombre de **campo** debe comenzar con una letra mayúscula.

* El comparador se coloca después del nombre de campo. Si se omite se asume que es Equal.  

  Comparadores: (Equal, NotEqual, GreaterThan, GreaterThanEqual, LessThan, LessThanEqual,Like)  

  Operadores Lógicos: (And, Or, Not)  


La anotación devuelve un valor de tipo Long.


Definición de la anotación:

 
```java

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface CountBy {

}

```
  


Ejemplo de uso:

 
```java
@Repository(entity = Oceano.class)
public interface OceanoRepository extends CrudRepository<Oceano, String>{ 
@CountBy
public Long countByIdOceanoAndOceanoNotEqualAndDate(String idoceano, String oceano, Date date);

@CountBy
public Long countByIdOceano(String idoceano);

@CountBy
public Long countByIdOceanoNotEqual(String idoceano);

@CountBy
public Long countByIdOceanoAndOceano(String idoceano, String oceano);

@CountBy
public Long countByIdOceanoAndOceanoNotFechaGreaterThan(String idoceano,String oceano, Date fecha);

@CountBy
public Long countByIdOceanoAndOceanoNotFechaOrActivo(String idoceano,String oceano, Date fecha, String activo);

@CountBy
public Long countByIdOceanoAndOceanoNotFechaOrActivoAndKm(String idoceano,String oceano, Date fecha, String activo, Integer km);

}

```
  



## @RegexCount


La anotación @RegexCount, cuenta el número de documentos que coinciden con una expresión regular especificada.

La anotación devuelve un valor de tipo Long.

Definición de la anotación:

 
```java

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface RegexCount {
    String where();
    CaseSensitive caseSensitive() default CaseSensitive.NO;
    LikeByType likeByType() default LikeByType.FROMTHESTART;
}

```  
  

En este ejemplo, se muestra un repositorio con dos métodos que cuentan los documentos basándose en una expresión regular para el campo 'oceano'. 
Al emplear CaseSensitive.NO  la expresión regular ignora las diferencias entre mayúsculas y minúsculas. Al establecer CaseSensitive.YES la expresión regular distingue entre mayúsculas y minúsculas :


```java

@RegexCount(where = "oceano .like. @oceano",caseSensitive = CaseSensitive.NO)
public Long regexCountOceano(String oceano);

@RegexCount(where = "oceano .like. @oceano", caseSensitive = CaseSensitive.YES)
public Long regexCountSensitive(String oceano);

    
``` 
     

## Pagination

La paginación se utiliza comúnmente junto con las anotaciones @Regex, @Query, @Lookup, @FindBy.

Cuando se use con la anotación @Query, es necesario incluir la sintaxis **.limit, .pagination, .skip y @pagination**. 

La clase Pagination contiene la siguiente definición:



```java

public class Pagination {
    private Integer page;
    private Integer size;

    public Pagination() {
    }

  //set/get
    
    private Integer next(){
       return page< size ? page++ :page;
    }
    
    public Integer skip(){
        return    page > 0 ? ((page - 1) * size) : 0;        
    }
    
    public Integer limit(){        
        return size;
    }    
  
}

```
 


Ejemplos del uso de pagination:

 
```java

@Query(where="pagination .skip. @pagination")
public List<Oceano> findAllPagination(Pagination pagination);

@Query(where = "oceano .eq. @oceano .limit. pagination .skip. @pagination .order. sorted .by. @sorted")
public List<Oceano> findByOceanoPagination(String oceano, Pagination pagination,Sorted sorted);

@Find()
public List<Oceano> findByIdOceanoNotEqualPagination(String idoceano,Pagination pagination);

```
 


## @Ping

La anotación @Ping crea un método que verifica la disponibilidad de la base de datos MongoDB a través de un 'ping'.

El método no requiere parámetros de entrada y devuelve un valor booleano. Como se muestra en la siguiente figura:


![](figura_04_01.png)

Definición de la anotación:

 
```java

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Ping {

}

```
 

Ejemplo de uso:

 
```java

  @Ping
  public Boolean ping();

```
 



El siguiente fragmento de código se implementa un endpoint utilizando [Microprofile](https://microprofile.io/) para verificar la conectividad con la base de datos.
 
```java

@GET
@Path("/ping")
public Boolean ping(){

    return oceanoRepository.ping().booleanValue();

}

```
 
Al compilar el proyecto, se generan las implementaciones de los métodos especificados en los repositorios.

La figura a continuación ilustra los resultados de la compilación:

![figura_04_02](figura_04_02.png)






## Resumen  

En este capítulo, se profundiza en la creación de repositorios para facilitar la interacción con bases de datos NoSQL. En el siguiente capítulo, se explora en detalle cómo Jmoordbcore gestiona atributos secuenciales o incrementales.