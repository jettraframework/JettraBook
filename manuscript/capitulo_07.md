# Capítulo 7


Este capítulo se centra en el uso del driver oficial Java para MongoDB, proporcionando ejemplos de código que ilustran cómo se utiliza el driver oficial para comunicarse con la base de datos.

Este capítulo incluye los temas:

* Java Drivers MongoDB

* MongoDBProducer

* ¿Qué es un EntitySupplier?

* Repositorios
 
* Save
 
* Update
 
* FindAll

* Paginación/Ordenación

* FindByPK

* Delete

* DeleteMany
 
* UpdateMany
 
* Filtros
 
* Query
 
* AutosecuenceRepository
 
* Ejemplo




## Java Drivers MongoDB

MongoDB proporciona un driver oficial de Java, que Jmoordb emplea para interactuar con la base de datos y realizar operaciones.


Es importante destacar que MongoDB proporciona dos drivers oficiales para su uso con aplicaciones Java:

* [Java Driver](https://www.mongodb.com/docs/drivers/java/sync/current) diseñado para aplicaciones síncronas.


* [Reactive Streams Driver](https://www.mongodb.com/docs/drivers/reactive-streams/) , destinado al procesamiento de streams asíncronos.


En los capítulos previos, se detallaron los pasos para configurar el entorno de trabajo en el archivo microprofile-config.properties. Es necesario definir las entidades y crear las interfaces de repositorio para cada una de ellas. 

En este capítulo, nos enfocaremos en el proyecto Maven denominado capitulo07.

Proceda a crear la clase MongoDBProducer, que permitirá interactuar con MongoDB desde nuestra aplicación Java. 

Esta conexión puede establecerse con una base de datos local o en un servidor cómo MongoDB Atlas. Independiente de la ubicación de la base de datos, se establecerá una comunicación básica.



## MongoDBProducer  

La conexión a la base de datos se configura en el archivo microprofile-config.properties, utilizando la propiedad **mongodb.uri** para especificar el URL de la base de datos.

A continuación, se presenta la configuración de la base de datos en el archivo microprofile-config.
 
```xml

#mongodb.uri=mongodb+srv://conexion-mongodb-atlas/;
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
mongodb.database2=practicadb

```
 
Proceda a crear la clase MongoDBProducer con el siguiente código:

 
```java


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
 

Vamos a profundizar el funcionamiento de las entidades y repositorios.


## ¿Qué es un EntitySupplier?


Las entidades en Jmoordbcore son clases Java, cuyos atributos representan elementos de un documento. Estos documentos pueden ser documentos embebidos y/o referenciados.

Un documento referenciado es una referencia tradicional que se busca en una colección referenciada utilizando la llave primaria. Solo se almacena el, **id** de la colección referenciada. 

Por ejemplo, la colección océano contiene un documento similar al siguiente:

```json

oceano{
      idoceano:"pacifico",
      oceano: "Oceano Pacifico"
}

```


Cree una entidad Java para representar los documentos de la colección 'oceano':

 
```java

@Entity()
public class Oceano {
    @Id
    private String idoceano;
    @Column
    private String oceano;
    @Column
    private Date fecha;

    public Oceano() {
    }

   //setter/getter
}

```
Durante la compilación del proyecto, Jmoordbcore genera automáticamente la clase OceanoSupplier.java en el mismo paquete donde se ha definido la entidad.

![](figura_07_00.png)

Los métodos generados incluyen:

* get public Oceano get(Supplier<? extends Oceano> s, Document document_ ) : Este método convierte un Document de MongoDB a la entidad correspondiente.

* getId(Supplier<? extends Oceano> s, Document document_, Boolean... showError): Este método devuelve un objeto del tipo de la entidad.


 
```java

@RequestScoped
public class OceanoSupplier  implements Serializable{

  public Oceano get(Supplier<? extends Oceano> s, Document document_, Boolean... showError) {
   Oceano oceano= s.get(); 
   Boolean show = true;
    try {
        if (showError.length != 0) {
            show = showError[0];
        }

        oceano.setIdoceano(document_.getString("idoceano"));
        oceano.setOceano(document_.getString("oceano"));

    } catch (Exception e) {
        if (show) {
           MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
        }
     }
     return oceano;
   }

  public Oceano getId(Supplier<? extends Oceano> s, Document document_, Boolean... showError) {
    Oceano oceano= s.get(); 
    Boolean show = true;
    try {
        if (showError.length != 0) {
            show = showError[0];
        }
        oceano.setIdoceano(document_.getString("idoceano"));
    } catch (Exception e) {
        if (show) {
           MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
       }
    }
     return oceano;
  }
  
```
* public Document toDocument(Oceano oceano) : Este método se utiliza para convertir una entidad 'Oceano' en un Document de MongoDB.

* public List<Document> toDocument(List<Oceano> oceanoList) : Este método convierte una lista de entidades 'Oceano' en una lista de Document de MongoDB.

```java
 public Document toDocument(Oceano oceano) {
       Document document_ = new Document();
       try {	 
           document_.put("idoceano",oceano.getIdoceano());
           document_.put("oceano",oceano.getOceano());	
       } catch (Exception e) {
          MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
         }
         return document_;
  }

  public List<Document> toDocument(List<Oceano> oceanoList) {
    List<Document> documentList_ = new ArrayList<>();
       try {	 
  	  for(Oceano oceano : oceanoList){
           Document document_ = new Document();
           document_.put("idoceano",oceano.getIdoceano());
           document_.put("oceano",oceano.getOceano());
           documentList_.add(document_);
          }
        } catch (Exception e) {
          MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
        }
        return documentList_;
   }

```


* Bson toUpdate(Oceano oceano): Este método genera un objeto Bson que contiene los atributos de la entidad oceano que se desea actualizar.

* List<Bson> toUpdate(List<Oceano> oceanoList): Este método devuelve una lista de objetos Bson, cada uno con los atributos de la entidad Oceano que se desea actualizar.


```java

public Bson toUpdate(Oceano oceano) {
   Bson update_ = Filters.empty();
   try {
     update_ = Updates.combine(	 
       Updates.set("idoceano",oceano.getIdoceano()),
       Updates.set("oceano",oceano.getOceano())	 
     );
    } catch (Exception e) {
     MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
    }
   return update_;
}

public List<Bson> toUpdate(List<Oceano> oceanoList) {
   List<Bson> bsonList_ = new ArrayList<>();
   try {
     for(Oceano oceano : oceanoList){
        Bson update_ = Filters.empty();
        update_ = Updates.combine(
                  Updates.set("idoceano",oceano.getIdoceano()),
                  Updates.set("oceano",oceano.getOceano())	
                  );
        bsonList_.add(update_);
        }
     } catch (Exception e) {
      MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
     }
     return bsonList_;
}

```

* Document toReferenced(Oceano oceano): Este método devuelve un objeto de tipo MongoDB Document que contiene el campo utilizado para hacer referencia a la entidad Oceano.

* List<Document> toReferenced(List<Oceano> oceanoList): Este método devuelve una lista de objetos de tipo MongoDB Document, cada uno contiene el campo utilizado para hacer referencia a una entidad de la lista oceanoList.

```java
    public Document toReferenced(Oceano oceano) {
        Document document_ = new Document();
        try {
            document_.put("idoceano",oceano.getIdoceano());
         } catch (Exception e) {
           MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
         }
         return document_;
     }

    public List<Document> toReferenced(List<Oceano> oceanoList) {
        List<Document> documentList_ = new ArrayList<>();
        try {	 
	 for(Oceano oceano : oceanoList){
	     Document document_ = new Document();
	     document_.put("idoceano",oceano.getIdoceano());
	     documentList_.add(document_);
          }
         } catch (Exception e) {
            MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
         }
         return documentList_;
     }


```



## Repositorios

En el capítulo 4, exploramos el uso de los repositorios en Jmoordbcore. Esta sección detalla el código que Jmoordbcore genera para las implementaciones de las interfaces de repositorio. 

Es recomendable que los repositorios hereden de **CrudRepository**. Esto proporciona al desarrollador los métodos para realizar las operaciones **CRUD**, minimizando la cantidad de métodos que deben ser escritos en la interfaz.

La interfaz **CrudRepository** incluye las siguientes definiciones de métodos:

```java

public interface CrudRepository<T extends Object, PK extends Object> {

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
  

Cuando se procesa la anotación @Repository, que hereda de CrudRepository, se genera automáticamente la implementación de los métodos definidos en la interfaz. El método findByPk(), también se utiliza en los **EntitySupplier** para buscar un documento por su llave primaria.

Cree una interfaz para su repositorio que herede de CrudRepository. Esta interfaz contendrá los métodos básicos para las operaciones CRUD, y puede incluir métodos adicionales según sea necesario.



```java

@Repository(entity = Oceano.class,jakartaSource = JakartaSource.JAKARTA)
public interface OceanoRepository extends CrudRepository<Oceano, String> {
    
    @Find
    public Optional<Oceano> findByIdoceano(String idoceano);
}


```


Cuando se procesa la anotación @Repository junto con CrudRepository, se genera automáticamente la implementación de los métodos definidos en la interfaz.

El método findByPk() se empleará también a través de los **EntitySupplier** para realizar búsquedas de un documento por su llave primaria.

En los métodos, MongoDatabase y MongoCollection se usan con frecuencia para referirse a la base de datos y a la colección respectivamente, que se obtienen del archivo microprofile-config.properties.

El código generado se explica a continuación:

* En la primera fase, se crea una clase con el alcance @ApplicationScoped que implementa la interfaz Repository.

* La segunda etapa implica la inyección de la configuración establecida en el archivo microprofile-config.properties, donde se obtiene el nombre de la base de datos y la colección. Estos tendrán prioridad sobre los valores establecidos en el archivo de configuración.

* En la tercera fase, se inyectan el repositorio, el manejo de autoincremento y la clase Supplier que corresponde a la entidad.


```java

@ApplicationScoped
public class OceanoRepositoryImpl  implements OceanoRepository{

    @Inject
    MongoClient mongoClient;
/**
* Microprofile Config
*/
    @Inject
    Config config;
    @Inject
    @ConfigProperty(name = "mongodb.database")
    String mongodbDatabase;

    String mongodbCollection = "oceano";
/**
* AutogeneratedRepository
*/
    @Inject
    com.avbravo.mongodbatlasdriver.repository.AutogeneratedRepository autogeneratedRepository;
/**
* Supplier
*/
    @Inject
    com.avbravo.mongodbatlasdriver.model.OceanoSupplier oceanoSupplier;

   private JmoordbException exception;
public JmoordbException getJmoordbException() {
   if(exception == null || exception.getLocalizedMessage()== null ){
    exception = new JmoordbException("");
   }
    return exception;
 }
public void setJmoordbException(JmoordbException exception) {    this.exception = exception; }

```


## Save

El método save() se emplea para almacenar un nuevo documento en una colección. Este devuelve un objeto de tipo Optional<> que permite verificar el resultado.

El proceso interno se realiza en los siguientes pasos:

1. Se accede a la base de datos utilizando mongoClient.getDatabase().

2. Se recuperan un objeto que representa la colección a través de database.getCollection().

3. Si la entidad utiliza una llave primaria autoincrementable, se genera un valor secuencial para la misma.

4. Al llamar al método findByPk(), se comprueba si existe un documento con una llave primaria similar.

5. Si no existe un documento con una llave primaria similar, se crea uno nuevo utilizando el método insertOne(). Note que se usa el método toDocument() para convertir una entidad a un Document de MongoDB.

Aquí un ejemplo del método save():

```java

@Override
public Optional<Oceano> save(Oceano oceano) {
  try {
     MongoDatabase database = mongoClient.getDatabase(mongodbDatabase);
     MongoCollection<Document> collection = database.getCollection(mongodbCollection);

     if (findByPk(oceano.getIdoceano()).isPresent()) { 
         MessagesUtil.warning("There is already a record with that id");
         exception = new JmoordbException("There is already a record with that id");
        return Optional.of(oceano);
     }

     InsertOneResult insertOneResult = collection.insertOne(oceanoSupplier.toDocument(oceano));
     if (insertOneResult.getInsertedId() != null) {
         return Optional.of(oceano);
     }
    } catch (Exception e) {
      MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
      exception = new JmoordbException(MessagesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
   }
     return Optional.empty();
 }

```


## Update


Para actualizar la entidad, se invoca al método updateOne() después de verificar la existencia de un documento. La llave primaria no se modifica, y se llama al método toUpdate() del Supplier, el cual devuelve un UpdateOptions con todas las condiciones de los atributos que se van a actualizar.

```java

@Override
public Boolean update(Oceano oceano) {
   try {
     MongoDatabase database = mongoClient.getDatabase(mongodbDatabase);
     MongoCollection<Document> collection = database.getCollection(mongodbCollection);

     if (!findByPk(oceano.getIdoceano()).isPresent()) { 
        MessagesUtil.warning("Not found a record with that id");
        exception = new JmoordbException("Not found a record with that id");
        return Boolean.FALSE;
     }

     Bson filter = Filters.empty();
     filter = Filters.eq("idoceano",oceano.getIdoceano());
     UpdateOptions options = new UpdateOptions().upsert(false);
     UpdateResult result = collection.updateOne(filter,oceanoSupplier.toUpdate(oceano),options);

     if (result.getModifiedCount() > 0) {
        return Boolean.TRUE;
     }
    } catch (Exception e) {
      MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
      exception = new JmoordbException(MessagesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
   }
   return Boolean.FALSE;
 }


```

## FindAll

Para consultar todos los documentos de la colección, empleamos el método find() sin ningún parámetro. Esto nos devuelve un objeto MongoCursor<Document>  que contiene los resultados de la consulta. Para transformar el Document en la entidad correspondiente, utilizamos el método get() proporcionado por OceanoSupplier.

```java

@Override
public List<Oceano> findAll() {
  List<Oceano> list = new ArrayList<>();
  try {

     MongoDatabase database = mongoClient.getDatabase(mongodbDatabase);
     MongoCollection<Document> collection = database.getCollection(mongodbCollection);

     MongoCursor<Document> cursor;
              cursor = collection.find()
                      .iterator();
     try{
        while (cursor.hasNext()) {
            list.add(oceanoSupplier.get(Oceano::new, cursor.next()));
        }
     } finally {
        cursor.close();
     } 
    } catch (Exception e) {
      MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
      exception = new JmoordbException(MessagesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
    }
   return list;

 }

```
  
  
 
## Paginación/Ordenación

Las clases Paginator y Sorted de Jmoordbcore se utilizan para implementar la paginación y ordenación en las consultas. Además, se usan los métodos skip(), limit() y sort() para su implementación.

```java

@Override
public List<Oceano> findAllPaginationSorted(Pagination pagination, Sorted sorted) {
    List<Oceano> list = new ArrayList<>();
    try {
      MongoDatabase database = mongoClient.getDatabase(mongodbDatabase);
      MongoCollection<Document> collection = database.getCollection(mongodbCollection);
      MongoCursor<Document> cursor;
               cursor = collection.find()
                       .skip(pagination.skip())
                       .limit(pagination.limit())
                       .sort(sorted.getSort())
                       .iterator();

      try{
        while (cursor.hasNext()) {
             list.add(oceanoSupplier.get(Oceano::new, cursor.next()));
        }
     } finally {
         cursor.close();
     } 
     } catch (Exception e) {
       MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
       exception = new JmoordbException(MessagesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
     }
     return list;

 }


```
        
        
## FindByPK

El método findByPk() lleva a cabo una consulta usando la llave primaria. Si no encuentra un resultado, devuelve Optional<> vacío. En caso contrario, retorna un Optional<Oceano> que contiene el resultado encontrado en la colección. Además, se utiliza el método get() de OceanoSupplier para transformar el resultado de Document a la entidad Oceano.
 
```java
@Override
public Optional<Oceano> findByPk(String id ) {
   try {
      MongoDatabase database = mongoClient.getDatabase(mongodbDatabase);
      MongoCollection<Document> collection = database.getCollection(mongodbCollection);

      Document doc = collection.find(eq("idoceano", id)).first();
      if(doc == null){
       return Optional.empty();
      }
     Oceano oceano = oceanoSupplier.get(Oceano::new, doc);
     return Optional.of(oceano);
    } catch (Exception e) {
      MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
      exception = new JmoordbException(MessagesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
    }
    return Optional.empty();
}
```


## Delete

El método deleteOne() se utiliza para eliminar un documento, devolviendo un valor Long que representa la cantidad de documentos que han sido eliminados.
 
```java

@Override
public Long deleteByPk(String id){
   try {
      MongoDatabase database = mongoClient.getDatabase(mongodbDatabase);
      MongoCollection<Document> collection = database.getCollection(mongodbCollection);

      MongoCursor<Document> cursor;
      Bson filter = Filters.eq("idoceano",id);
      com.mongodb.client.result.DeleteResult deleteResult = collection.deleteOne(filter);

      return deleteResult.getDeletedCount();
    } catch (Exception e) {
      MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
      exception = new JmoordbException(MessagesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
   }
   return 0L;
 }

```

## DeleteMany

El método deleteMany() se emplea para eliminar documentos que satisfacen las condiciones definidas en el parámetro Search. Este método devuelve un valor de tipo Long, que representa el número total de documentos eliminados.

```java

@Override
public Long deleteMany(com.jmoordb.core.model.Search search){
    try {
      MongoDatabase database = mongoClient.getDatabase(mongodbDatabase);
      MongoCollection<Document> collection = database.getCollection(mongodbCollection);

      Document whereCondition = new Document();
      whereCondition = search.getFilter();
      com.mongodb.client.result.DeleteResult deleteResult = collection.deleteMany(whereCondition);

      return deleteResult.getDeletedCount();
     } catch (Exception e) {
       MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
       exception = new JmoordbException(MessagesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
     }
    return 0L;
 }


```



## UpdateMany

Para actualizar varios documentos simultáneamente, use el método updateMany. Este devolverá un valor Long que representa el número de documentos modificados, según el filtro aplicado en el parámetro Bson query.

```java

@Override
public Long updateMany(Bson query, Bson update) {
   try {
     MongoDatabase database = mongoClient.getDatabase(mongodbDatabase);
     MongoCollection<Document> collection = database.getCollection(mongodbCollection);
     UpdateResult result = collection.updateMany(query, update);
     return result.getModifiedCount();
    } catch (Exception e) {
      MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
      exception = new JmoordbException(MessagesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
    }
   return 0L;
 }

```


  
## Builder/Filter

MongoDB ofrece la clase [Builder](https://www.mongodb.com/docs/drivers/java/sync/current/fundamentals/builders/), que permite construir métodos compuestos por una o más condiciones.

Por ejemplo, para realizar una consulta de personas que practican baloncesto y cuya edad sea menor de 35 años, y que el documento devuelto excluya el id, pero incluya el nombre, se podría utilizar Builder de la siguiente manera:

```java

Bson filter = and(eq("deporte", "baloncesto"), lt("edad", 35));

Bson projection = fields(excludeId(), include("nombre"));

collection.find(filter).projection(projection);

```


La clase [Filter](https://www.mongodb.com/docs/drivers/java/sync/current/fundamentals/builders/filters/#std-label-filters-builders) proporciona métodos con operadores que facilitan la creación de consultas. Estas consultas devuelven una instancia de tipo BSON, que luego es convertida a una entidad.

Por favor, incorpore el siguiente método en OceanoRepository:

  
```java

@Find()
public List<Oceano> findByIdOceanoAndOceanoNotFecha(String idoceano, String oceano, Date fecha);

```
  
Una vez compilado el proyecto, observe la clase generada OceanoRepositoryImpl.java y busque el método findByIdOceanoAndOceanoNotFecha. Este método contiene un filtro de tipo BSON que establece las condiciones para realizar la búsqueda.

Es importante recordar que JmoordbCore asume que las búsquedas de fechas serán de tipo @ExcludeTime, excluyendo así las horas y minutos. Sin embargo, si desea incluir las horas y minutos en la consulta, puede utilizar @IncludeTime.

```java

@Override
public java.util.List<com.avbravo.mongodbatlasdriver.model.Oceano> findByIdOceanoAndOceanoNotFecha(java.lang.String idoceano,java.lang.String oceano,java.util.Date fecha) {
   List<Oceano> list = new ArrayList<>();
   try {
     MongoDatabase database = mongoClient.getDatabase(mongodbDatabase);
     MongoCollection<Document> collection = database.getCollection(mongodbCollection);
     MongoCursor<Document> cursor;

     Bson filter =Filters.and(
               Filters.eq("idOceano",idoceano)
               ,Filters.eq("oceano",oceano)
               ,Filters.not(
                   Filters.and(
                     Filters.gte("and",JmoordbCoreDateUtil.dateToLocalDateTimeFirstHourOfDay(fecha)),
                    Filters.lte("and",JmoordbCoreDateUtil.dateToLocalDateTimeLastHourOfDay(fecha))
                )
               )
       );

       cursor = collection.find(filter).iterator();
       try{
           while (cursor.hasNext()) {
             list.add(oceanoSupplier.get(Oceano::new, cursor.next()));
           }
        } finally {
           cursor.close();
       } 
    } catch (Exception e) {
      MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
      exception = new JmoordbException(MessagesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
     }
    return list;

 }

```



## Query

Vamos a definir el método utilizando la anotación @Query, e incluiremos la búsqueda con horas y minutos usando @IncludeTime:

```java

@Query(where = "idoceano .eq. @idoceano .and. oceano .eq. @oceano .not. fecha .gt. @fecha")
public List<Oceano> queryByIdOceanoAndOceanoNotFecha(String idoceano, String oceano, @IncludeTime Date fecha)

```
  


Jmoordbcore, procesa las anotaciones, verifica la coincidencia de los parámetros con los definidos en la anotación **@Query** y genera los filtros basándose en estas condiciones. 

A continuación, se muestra un segmento de código generado donde se observa que la búsqueda por fecha ha cambiado:

 
```java

@Override
public java.util.List<com.avbravo.mongodbatlasdriver.model.Oceano> queryByIdOceanoAndOceanoNotFecha(java.lang.String idoceano,java.lang.String oceano,java.util.Date fecha) {
   List<Oceano> list = new ArrayList<>();
   try {
     MongoDatabase database = mongoClient.getDatabase(mongodbDatabase);
     MongoCollection<Document> collection = database.getCollection(mongodbCollection);
     MongoCursor<Document> cursor;
     Bson filter =Filters.and(
               Filters.eq("idoceano",idoceano)
              ,Filters.eq("oceano",oceano)
              ,Filters.not(
                    Filters.gt("fecha",fecha)
                    )
      );
    cursor = collection.find( filter ).iterator();
    try{
        while (cursor.hasNext()) {
           list.add(oceanoSupplier.get(Oceano::new, cursor.next()));
        }
    } finally {
       cursor.close();
   } 
   } catch (Exception e) {
        MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
        exception = new JmoordbException(MessagesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
  }
  return list;

 }


```



## AutosecuenceRepository

Como recordará, MongoDB no admite campos incrementales o valores secuenciales. Para implementarlo, utilizamos una interfaz de repositorio que nos permite gestionarlo de manera sencilla, tal como se muestra a continuación.

```java

@AutosecuenceRepository(entity = Autosequence.class, jakartaSource = JakartaSource.JAKARTA, database = "{mongodb.database}", collection = "autoincrementable")
public interface AutogeneratedRepository {

@Autogenerated() 
public Long generate(String database,String collection);

}

```

Cuando se compila el proyecto, se genera la implementación de la interfaz y los métodos generados se presentan a continuación:




```java

@ApplicationScoped


public class  AutogeneratedRepositoryImpl  implements AutogeneratedRepository {

   @Inject 
   MongoClient mongoClient;
/*
   Microprofile Config
*/
   @Inject 
   Config config;
   @Inject 
   @ConfigProperty(name = "mongodb.database")
   String mongodbDatabase;

   String mongodbCollection = "autoincrementable";

@Override
public Long generate(String database, String collection) {
   try {
     Optional<Autosequence> autosequenceOptional = findById(database + "_" + collection);
     if (!autosequenceOptional.isPresent()) {
          Long l = Long.valueOf("0");
        Autosequence autosequence = new Autosequence(database + "_" + collection, l);
        save(autosequence);
     }
     Optional<Autosequence> autosequenceIncrementOptional = findOneAndUpdate(database + "_" + collection);
     if (!autosequenceIncrementOptional.isPresent()) {
        return -1L;
     } else {
        return autosequenceIncrementOptional.get().getSecuence();
     }
   } catch (Exception e) {
     MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
   }
  return -1L;
}

```

El método 'generate' verifica si existe un registro para esa colección. Si no existe, genera un valor inicial.

Si el registro existe, se utiliza el método findOneAndUpdate() para obtener el valor, incrementarlo y devolver el resultado en una única operación:


```java

public Optional<Autosequence> findOneAndUpdate(String databasecollection) {
   try {
     Long  increment = new Long(1);            // Integer increment = 1;
     Document doc = new Document("databasecollection", databasecollection);
     Document inc = new Document("$inc", new Document("sequence", increment));
     FindOneAndUpdateOptions findOneAndUpdateOptions = new FindOneAndUpdateOptions();
     findOneAndUpdateOptions.upsert(true);
     findOneAndUpdateOptions.returnDocument(ReturnDocument.AFTER);
     MongoDatabase database = mongoClient.getDatabase(mongodbDatabase);
     MongoCollection<Document> collection = database.getCollection(mongodbCollection);
     Document iterable = collection.findOneAndUpdate(doc, inc, findOneAndUpdateOptions);

     Autosequence autosequence = new Autosequence(databasecollection, iterable.getLong("sequence"));
    //Autosequence autosequence = get(Autosequence::new, iterable);
     return Optional.of(autosequence);
   } catch (Exception e) {
     MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
   }
  return Optional.empty();
}


```

El método  get() retornará un objeto de tipo Autosequence.

```java
public Autosequence get(Supplier<? extends Autosequence> s, Document document) {
  Autosequence autosequence = s.get();
   try {
         autosequence.setSecuence(document.getLong("secuence"));
         autosequence.setDatabasecollection(document.getString("databasecollection"));
   } catch (Exception e) {
     MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
   }
  return autosequence;
}

```

El método save() se emplea para almacenar en la base de datos la secuencia inicial de valores.

```java

public Optional<Autosequence> save(Autosequence autosequence) {
   try {
     MongoDatabase database = mongoClient.getDatabase(mongodbDatabase);
     MongoCollection<Document> collection = database.getCollection(mongodbCollection);
     InsertOneResult insertOneResult = collection.insertOne(Document.parse(autosequence.toJson(autosequence)));
     if (insertOneResult.getInsertedId() != null) {
        return Optional.of(autosequence);
     }
   } catch (Exception e) {
     MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
   }
  return Optional.empty();
}

```

El método findById() verífica la existencia de un documento específico en la colección.


```java

public Optional<Autosequence> findById(String databasecollection) {
   try {
     MongoDatabase database = mongoClient.getDatabase(mongodbDatabase);
     MongoCollection<Document> collection = database.getCollection(mongodbCollection);
     Document doc = collection.find(eq("databasecollection", databasecollection)).first();
     Autosequence autosequence = get(Autosequence::new, doc);
     return Optional.of(autosequence);
   } catch (Exception e) {
     MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
   }
  return Optional.empty();
}


}


```


En la sección siguiente, se muestra el documento JSON que se ha generado para gestionar la secuencial de una clave primaria.

```json
{
  "databasecollection" : "database_collection",
  "sequence" : NumberLong(1)
}

```





## Ejemplo 

Cree una entidad Planeta, en la cual se aplicará la anotación @Id(strategy = GenerationType.AUTO) al campo que se utilizará como clave primaria.

```java

@Entity
public class Planeta {

    @Id(strategy = GenerationType.AUTO)
    private Long idplaneta;
    
    @Column 
    private String planeta;
    
    @Referenced(from = "oceano", localField = "idoceano",typeReferenced = TypeReferenced.REFERENCED)
    private List<Oceano> oceano;

//set/get
}

```

Establezca el repositorio correspondiente para la entidad Planeta:

```java

@Repository(entity = Planeta.class)
public interface PlanetaRepository extends CrudRepository<Planeta, Long> {
    

}


```

Durante la compilación del proyecto, se genera la clase PlanetaSupplier. Notará que tanto el repositorio como el supplier de la entidad referenciada son inyectados en esta clase:


```java

@RequestScoped
public class PlanetaSupplier  implements Serializable{

   @Inject
   com.avbravo.mongodbatlasdriver.repository.OceanoRepository oceanoRepository ;
   @Inject
   OceanoSupplier oceanoSupplier ;


```

La referencia a Oceano se maneja como una lista de tipo List<Oceano>. Para buscar en la colección referenciada, iteramos sobre la lista utilizando el método get(). 

En los siguientes capítulos, se mostrará el uso de @ViewReferenced que facilita la realización de búsquedas en revistas referenciadas.

```java

public Planeta get(Supplier<? extends Planeta> s, Document document_, Boolean... showError) {
    Planeta planeta= s.get(); 
 Boolean show = true;
 try {
     if (showError.length != 0) {
          show = showError[0];
     }
     planeta.setIdplaneta(document_.getLong("idplaneta"));
     planeta.setPlaneta(document_.getString("planeta"));
    // Referenced List<oceano>
     List<Document> oceanoDocumentList = (List)document_.get("oceano");
     List<Oceano> oceanoList = new ArrayList<>();
     for( Document oceanoDoc :oceanoDocumentList){
          Oceano oceano = oceanoSupplier.getId(Oceano::new,oceanoDoc);
          Optional<Oceano> oceanoOptional = oceanoRepository.findByPk(oceano.getIdoceano());
          if(oceanoOptional.isPresent()){
             oceanoList.add(oceanoOptional.get());
          }
      }
    planeta.setOceano(oceanoList);
  } catch (Exception e) {
    if (show) {
      MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
    }
 }
 return planeta;
 }

```

Para convertir una entidad a un documento utilizando el método toDocument, simplemente necesitamos llamar al método toReferenced() de la entidad que está siendo referenciada.

```java

public Document toDocument(Planeta planeta) {
    Document document_ = new Document();
    try {
         document_.put("idplaneta",planeta.getIdplaneta());
         document_.put("planeta",planeta.getPlaneta());
    // Referenced List<oceano>
         document_.put("oceano",oceanoSupplier.toReferenced(planeta.getOceano()));
     } catch (Exception e) {
        MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
     }
     return document_;
 }

```

Revise la clase PlanetaRepositoryImpl.java, donde se utiliza AutogeneratedRepository para la implementación de generación automática de valores secuenciales.

```java

    @Inject
    com.avbravo.mongodbatlasdriver.repository.AutogeneratedRepository autogeneratedRepository;

```

En el método save() llama al método generate() de AutogeneratedRepository, que devuelve un valor autoincrementado para la clave primaria. Aquí está el segmento de código correspondiente:

```java

planeta.setIdplaneta(autogeneratedRepository.generate(mongodbDatabase, mongodbCollection));

```

Este segmento es parte del método save() e incluye una verificación adicional para asegurar la correcta secuencia.


```java

 @Override
    public Optional<Planeta> save(Planeta planeta) {
        try {
               MongoDatabase database = mongoClient.getDatabase(mongodbDatabase);
               MongoCollection<Document> collection = database.getCollection(mongodbCollection);
               	Boolean success = Boolean.FALSE;
	while (!success) {
               planeta.setIdplaneta(autogeneratedRepository.generate(mongodbDatabase, mongodbCollection));
               if (findByPk(planeta.getIdplaneta()).isPresent()) { 
                   MessagesUtil.warning("There is already a record with that id");
                    exception = new JmoordbException("There is already a record with that id");
                               }
               	else{
		  success= Boolean.TRUE;
	     }
	   }
               InsertOneResult insertOneResult = collection.insertOne(planetaSupplier.toDocument(planeta));
               if (insertOneResult.getInsertedId() != null) {
                  return Optional.of(planeta);
               }
         } catch (Exception e) {
              MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
               exception = new JmoordbException(MessagesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
         }
         return Optional.empty();
}

```

Para explorar todos los métodos generados, compile el proyecto de ejemplo que es incluido con este libro.  


## Resumen 

En este capítulo se explica el código que genera Jmoordbcore para el driver oficial de Java para MongoDB. 

En el siguiente capítulo, se explorará el proceso para gestión de documentos embebidos y referenciados.
