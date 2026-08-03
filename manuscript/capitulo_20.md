# Capítulo 20


En este capítulo, vamos a explorar cómo utilizar llaves primarias de tipo ObjectId y UUID.

Este capítulo incluye los temas:


* Llaves primarias ObjectId


* LLaves primarias de tipo UUID.


## Llaves primarias ObjectId

En MongoDB cada documento creado requiere un unico campo _id, que actua como llave primaria. Este campo es generado mediante ObjectId. Los campos ObjectId tienen una longitud de 12 bytes compuestos de la siguuiente manera:


* Los primeros 4 bytes, representan la creación del ObjectId, medida en segundos desde la época Unix.

* Los siguientes 5 bytes , reprensentan un valor aleatorio es único para la máquina (3 bytes) y 2 bytes para el proceso.

* Lo ultimos  3 bytes, es un  valor aleatorio.



Por ejemplo:

```json
{
    _id: ObjectId('669c8974e08aa75b1c303ff1'),
    pais:"Panama",
    code:"507"
}

```

Como recordara en el capitulo 3 se menciono los tipos de generación de valores para llaves primarias:


* @Id(strategy =  GenerationType.AUTO)  : Activa el autoincrementable, solo para atributos de tipo Long.

* @Id(strategy =  GenerationType.NONE) : Predeterminado, no usa autoincrementable.

* @Id(strategy = GenerationType.OBJECTID) : Utiliza el atributo _id de MongoDB como campo llave.

* @Id(strategy = GenerationType.UUID) : Utiliza genera valores UUID para el campo llave.


Reglas de uso:

* Para los llaves primarias declarados como ObjectId se debe utilizar GenerationType.OBJECTID con @Id.

* El atributo debe llamarse _id para que coincida con el campo almacenado en el documento de MongoDB.

* Se pueden utilizar llaves primarias de tipo ObjectId con las anotaciones @Entity, @ViewEntity pero no con documentos embebidos (@Embeddable).

```java
@Id
ObjectId _id;

```

* Se pueden definir columnas de tipo ObjectId con las anotaciones @Entity, @ViewEntity pero no con documentos embebidos(@Embeddable).

```java

@Column
private ObjectId _id;

```

* Tenga presente que al definir un nuevo campo de tipo ObjectId este sera generado por MongoDB y mediante JmoordbCore se accede al valor generado y es asignado al atributo _id de la entidad especificada.


* Para convertir un String a ObjectId utilice

```java
 ObjectId id =new ObjectId(stringvalue);

```

* Para convertir un ObjectId to String utilice

```java
ObjectId id =new ObjectId(stringvalue);

id.toString();

```

```java

import com.jmoordb.core.annotation.Column;
import com.jmoordb.core.annotation.Entity;
import com.jmoordb.core.annotation.Id;
import com.jmoordb.core.annotation.enumerations.GenerationType;
import java.util.Objects;
import org.bson.types.ObjectId;
@Entity
public class Persona {
    @Id(strategy = GenerationType.OBJECTID )
    private ObjectId _id;
    @Column
    private String nombre;

    public Persona() {
    }

    public Persona(ObjectId _id, String nombre) {
        this._id = _id;
        this.nombre = nombre;
    }

    public ObjectId getId() {
        return _id;
    }

    public void setId(ObjectId _id) {
        this._id = _id;
    }

    public String getNombre() {
        return nombre;
    }

    public void setNombre(String nombre) {
        this.nombre = nombre;
    }

    @Override
    public int hashCode() {
        int hash = 5;
        hash = 79 * hash + Objects.hashCode(this._id);
        hash = 79 * hash + Objects.hashCode(this.nombre);
        return hash;
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) {
            return true;
        }
        if (obj == null) {
            return false;
        }
        if (getClass() != obj.getClass()) {
            return false;
        }
        final Persona other = (Persona) obj;
        if (!Objects.equals(this.nombre, other.nombre)) {
            return false;
        }
        return Objects.equals(this._id, other._id);
    }

    @Override
    public String toString() {
        return "Persona{" + "_id=" + _id + ", nombre=" + nombre + '}';
    }
    

   
    
}

```

Cree el repositorio

```java
import com.estacionesserver.model.Persona;
import com.jmoordb.core.annotation.repository.Count;
import com.jmoordb.core.annotation.repository.Lookup;
import com.jmoordb.core.annotation.repository.Repository;
import com.jmoordb.core.model.Search;
import com.jmoordb.core.repository.CrudRepository;
import java.util.List;
import org.bson.types.ObjectId;

/**
 *
 * @author avbravo
 */
@Repository(database = "{mongodb.database}", entity = Persona.class)
//public interface MedicionRepository extends CrudRepository<Medicion, Long> {
public interface PersonaRepository extends CrudRepository<Persona, ObjectId> {

    @Lookup
    public List<Persona> lookup(Search search);

    @Count()
    public Long count(Search... search);    
    
}


```

cree el controller

```java
@Path("persona")
@RequestScoped
public class PersonaController implements Serializable {


    @Inject
    PersonaRepository medicionRepository;


    @GET 
    public List<Persona> findAll() {
         return medicionRepository.findAll();
    }

    @GET 
    @Path("id")
    public Persona findById(@QueryParam("id") String id) {
         return medicionRepository.findByPk(new ObjectId(id)).orElseThrow(
                () -> new WebApplicationException("No hay medicion con idmedicion " + id, Response.Status.NOT_FOUND));

    }
    @POST
    public Response save(
            @RequestBody(description = "Crea una persona.", content = @Content(mediaType = "application/json", schema = @Schema(implementation = Persona.class))) Persona medicion) {
        Optional<Persona> medicionOptional = medicionRepository.save(medicion);
        if (medicionOptional.isPresent()) {

            return Response.status(201).entity(medicionOptional.get()).build();
        } else {
            return Response.status(400).entity("Error " + medicionRepository.getJmoordbException().getLocalizedMessage()).build();
        }

    }

    @PUT
    public Response update(
            @RequestBody(description = "Actualiza persona", content = @Content(mediaType = "application/json", schema = @Schema(implementation = Persona.class))) Persona medicion) {
        
        if (medicionRepository.update(medicion)) {
            return Response.status(201).entity(medicion).build();
        } else {
            return Response.status(400).entity("Error " + medicionRepository.getJmoordbException().getLocalizedMessage()).build();
        }
    }

    @DELETE
    @Path("id")
      public Response delete(@QueryParam("id") String id) {
           
        if (medicionRepository.deleteByPk(new ObjectId(id)) == 0L) {
            return Response.status(201).entity(Boolean.TRUE).build();
        } else {
            return Response.status(400).entity("Error " + medicionRepository.getJmoordbException().getLocalizedMessage()).build();
        }
    }

    @DELETE
    @Path("deletemany")
    public Response deleteMany(@QueryParam("filter") String filter, @QueryParam("idestacion") Long idestacion, @QueryParam("anio") Integer anio, @QueryParam("mes") Integer mes) {
       
        Search search = DocumentUtil.convertForLookup(filter, "", 0, 0);
        if (medicionRepository.deleteMany(search) == 0L) {
            return Response.status(201).entity(Boolean.TRUE).build();
        } else {
            return Response.status(400).entity("Error " + medicionRepository.getJmoordbException().getLocalizedMessage()).build();
        }
    }
 
    @GET
    @Path("lookup")
    @Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})
    public List<Persona> lookup(@QueryParam("filter") String filter, @QueryParam("sort") String sort, @QueryParam("page") Integer page, @QueryParam("size") Integer size, @QueryParam("idestacion") Long idestacion, @QueryParam("anio") Integer anio, @QueryParam("mes") Integer mes) {
     
        List<Persona> suggestions = new ArrayList<>();
        try {

            Search search = DocumentUtil.convertForLookup(filter, sort, page, size);
          
            suggestions = medicionRepository.lookup(search);

        } catch (Exception e) {

            MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + "error: " + e.getLocalizedMessage());
        }
        System.out.println("Resultado: " + suggestions);
        return suggestions;
    }

    
    @GET
    @Path("count")
    @Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})
    public Long count(@QueryParam("filter") String filter, @QueryParam("sort") String sort, @QueryParam("page") Integer page, @QueryParam("size") Integer size, @QueryParam("idestacion") Long idestacion, @QueryParam("anio") Integer anio, @QueryParam("mes") Integer mes) {
        Long result = 0L;
        try{
            Search search = DocumentUtil.convertForLookup(filter, sort, page, size);
            result = medicionRepository.count(search);

        } catch (Exception e) {
            MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + "error: " + e.getLocalizedMessage());
        }

        return result;
    }

        
}

```

Ejecute el proyecto


```shell

mvn clean verify payara-micro:start

```


Obtenemos una salida como

```shell

Payara Micro URLs:
http://localhost:8080/capitulo20

'capitulo20' REST Endpoints:
GET     /capitulo20/api/application.wadl
POST    /capitulo20/api/persona
PUT     /capitulo20/api/persona
GET     /capitulo20/api/persona/count
DELETE  /capitulo20/api/persona/deletemany
DELETE  /capitulo20/api/persona/id
GET     /capitulo20/api/persona/id
GET     /capitulo20/api/persona/lookup

]]


```




Ejecute mediante curl para insertar una nueva persona

```shell

curl --location --request POST 'http://localhost:8080/capitulo20/api/persona/' --header 'Content-Type: application/json' --data-raw '{ "nombre": "Shadia"}'


```

Nos devuelve de respuesta un JSON correspondiente al documento insertado en la colección Persona



```json

{"id":{"date":"2024-07-22T02:59:35Z[UTC]","timestamp":1721617175},"nombre":"Shadia"}

```



Para consultarlo  mediante curl

```shell

curl --location --request GET http://localhost:8080/capitulo20/api/persona

```

Nos devuelve

```json

{"id":{"date":"2024-07-22T02:59:35Z[UTC]","timestamp":1721617175},"nombre":"Shadia"}

```

Como puede observar se genera el id como un documento embebido con los atributos date y timestamp.




En cambio al el documento desde la consola de MongoDB con

```shell

 use capitulo20db

 db.persona.find().pretty()

```





Obtendremos la salida como se muestra a continuación con ObjectId('669dcf3a5a624a2f2eb68482').

```json

[ 
{ _id: ObjectId('669dcf3a5a624a2f2eb68482'), nombre: 'Shadia' } 
]

```


Realizar la consulta mediante el ObjectId

```shell
curl --location --request GET http://localhost:8080/capitulo20/api/persona/id?id=669dcf3a5a624a2f2eb68482

```

Nos devuelve

```json
{"id":{"date":"2024-07-22T03:17:14Z[UTC]","timestamp":1721618234},"nombre":"Shadia"}

```

### Revisar esto
Realizar la consulta mediante el ObjectId

```shell
curl --location --request GET http://localhost:8080/capitulo20/api/persona/fechatimestamp?fecha=2024-07-22T03:17:14Z&timestamp=1721618234

```


### ObjectId con @Columun


Defina una entidad llamada Pais con una columna de tipo ObjectId. Recuerde que @DocummentEmbbedable no soporta el uso de ObjectId para columnas ni para campos llave.


```java
import com.jmoordb.core.annotation.Column;
import com.jmoordb.core.annotation.Entity;
import com.jmoordb.core.annotation.Id;
import com.jmoordb.core.annotation.enumerations.GenerationType;
import java.util.Objects;
import org.bson.types.ObjectId;

/**
 *
 * @author avbravo
 */
@Entity
public class Pais {
    @Id(strategy = GenerationType.AUTO)
    private Long idpais;
    @Column
    private String pais;
    
    @Column
    private ObjectId _id;

    public Pais() {
    }
...
}


```


Declare un repositorio

```java
@Repository(entity = Pais.class, jakartaSource = JakartaSource.JAKARTA)
public interface PaisRepository extends CrudRepository<Pais, Long> {

  
}


```

Cree el controller


```java
@Path("pais")
public class PaisController {

    @Inject
    PaisRepository paisRepository;

    @GET
    @Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})
    public List<Pais> findAll() {

        return paisRepository.findAll();
    }

    @GET
    @Path("{idpais}")
    public Pais findByIdpais(
            @Parameter(description = "El idpais", required = true, example = "1", schema = @Schema(type = SchemaType.NUMBER)) @PathParam("idpais") Long idpais) {

        return paisRepository.findByPk(idpais).orElseThrow(
                () -> new WebApplicationException("No hay pais con idpais " + idpais, Response.Status.NOT_FOUND));

    }

    @POST
    public Response save(
            @RequestBody(description = "Crea un nuevo pais.", content = @Content(mediaType = "application/json", schema = @Schema(implementation = Pais.class))) Pais pais) {

        return Response.status(Response.Status.CREATED).entity(paisRepository.save(pais)).build();
    }

    @PUT
    public Response update(
            @RequestBody(description = "Crea un nuevo pais.", content = @Content(mediaType = "application/json", schema = @Schema(implementation = Pais.class))) Pais pais) {

        return Response.status(Response.Status.CREATED).entity(paisRepository.update(pais)).build();
    }

    @DELETE
    @Path("{idpais}")
    public Response delete(
            @Parameter(description = "El elemento idpais", required = true, example = "1", schema = @Schema(type = SchemaType.NUMBER)) @PathParam("idpais") Long idpais) {
        paisRepository.deleteByPk(idpais);
        return Response.status(Response.Status.NO_CONTENT).build();
    }

}


```



Ejecute el proyecto

```shell
mvn clean verify payara-micro:start

```


Generar un nuevo documento de pais mediante

```shell

curl --location --request POST 'http://localhost:8080/capitulo20/api/pais/' --header 'Content-Type: application/json' --data-raw '{ "pais": "Panama"}'


```

Se puede observa la respuesta que genere el ObjectId para la columna.

```json
{"id":{"date":"2024-07-22T16:42:46Z[UTC]","timestamp":1721666566},"idpais":5,"pais":"Panama"}

```


## LLaves primarias de tipo UUID.


Para utilizar UUID como llave primaria.

Reglas:

* Para gener un UUID utilice

```java
   UUID uuid = UUID.randomUUID();
   String uuidAsString = uuid.toString();

```

* Convertir un String a UUID utilice

```java
idUuidMongo.setId(UUID.fromString(document_.getString("id")));

```



Crear la entidad


```java

@Entity
public class Equipo {

    @Id(strategy = GenerationType.UUID)
    UUID uid;
    @Column
    private String equipo;
...

}

```


Cree el repositorio

```java
@Repository(database = "{mongodb.database}", entity = Equipo.class)
public interface EquipoRepository extends CrudRepository<Equipo,UUID> {

    @Lookup
    public List<Equipo> lookup(Search search);

    @Count()
    public Long count(Search... search);
    
  
    
    
}


```

Cree el controller observe que:

* findById(@QueryParam("id") String id) recibe un String y lo convierte a UUID
* delete(@QueryParam("id") String id) recibe un String y lo convierte a UUID


```java

@Path("equipo")
@RequestScoped
public class EquipoController implements Serializable {


 
    @Inject
    EquipoRepository equipoRepository;


    @GET 
    public List<Equipo> findAll() {
         return equipoRepository.findAll();
    }

    @GET 
    @Path("id")
    public Equipo findById(@QueryParam("id") String id) {
            UUID _uuid=    UUID.fromString(id);
         return equipoRepository.findByPk(_uuid).orElseThrow(
                () -> new WebApplicationException("No hay equipo con idequipo " + id, Response.Status.NOT_FOUND));

    }
    
    
   
    
    
    @POST
    public Response save(
            @RequestBody(description = "Crea una equipo.", content = @Content(mediaType = "application/json", schema = @Schema(implementation = Equipo.class))) Equipo equipo) {
        Optional<Equipo> equipoOptional = equipoRepository.save(equipo);
        if (equipoOptional.isPresent()) {

            return Response.status(201).entity(equipoOptional.get()).build();
        } else {
            return Response.status(400).entity("Error " + equipoRepository.getJmoordbException().getLocalizedMessage()).build();
        }

    }

    @PUT
    public Response update(
            @RequestBody(description = "Actualiza equipo", content = @Content(mediaType = "application/json", schema = @Schema(implementation = Equipo.class))) Equipo equipo) {
        
        if (equipoRepository.update(equipo)) {
            return Response.status(201).entity(equipo).build();
        } else {
            return Response.status(400).entity("Error " + equipoRepository.getJmoordbException().getLocalizedMessage()).build();
        }
    }

    @DELETE
    @Path("id")
      public Response delete(@QueryParam("id") String id) {
             UUID _uuid=    UUID.fromString(id);
        if (equipoRepository.deleteByPk(_uuid) == 0L) {
            return Response.status(201).entity(Boolean.TRUE).build();
        } else {
            return Response.status(400).entity("Error " + equipoRepository.getJmoordbException().getLocalizedMessage()).build();
        }
    }

    @DELETE
    @Path("deletemany")
    public Response deleteMany(@QueryParam("filter") String filter, @QueryParam("idestacion") Long idestacion, @QueryParam("anio") Integer anio, @QueryParam("mes") Integer mes) {
       
        Search search = DocumentUtil.convertForLookup(filter, "", 0, 0);
        if (equipoRepository.deleteMany(search) == 0L) {
            return Response.status(201).entity(Boolean.TRUE).build();
        } else {
            return Response.status(400).entity("Error " + equipoRepository.getJmoordbException().getLocalizedMessage()).build();
        }
    }
 
    @GET
    @Path("lookup")
    @Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})
    public List<Equipo> lookup(@QueryParam("filter") String filter, @QueryParam("sort") String sort, @QueryParam("page") Integer page, @QueryParam("size") Integer size, @QueryParam("idestacion") Long idestacion, @QueryParam("anio") Integer anio, @QueryParam("mes") Integer mes) {
     
        List<Equipo> suggestions = new ArrayList<>();
        try {

            Search search = DocumentUtil.convertForLookup(filter, sort, page, size);
          
            suggestions = equipoRepository.lookup(search);

        } catch (Exception e) {

            MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + "error: " + e.getLocalizedMessage());
        }
        System.out.println("Resultado: " + suggestions);
        return suggestions;
    }

    
    @GET
    @Path("count")
    @Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})
    public Long count(@QueryParam("filter") String filter, @QueryParam("sort") String sort, @QueryParam("page") Integer page, @QueryParam("size") Integer size, @QueryParam("idestacion") Long idestacion, @QueryParam("anio") Integer anio, @QueryParam("mes") Integer mes) {
        Long result = 0L;
        try{
            Search search = DocumentUtil.convertForLookup(filter, sort, page, size);
            result = equipoRepository.count(search);

        } catch (Exception e) {
            MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + "error: " + e.getLocalizedMessage());
        }

        return result;
    }

        
}


```

Generar un nuevo documento de equipo

```shell

curl --location --request POST 'http://localhost:8080/capitulo20/api/equipo/' --header 'Content-Type: application/json' --data-raw '{ "equipo": "Barcelona F.C."}'


```

Para consultarlo  mediante curl

```shell

curl --location --request GET http://localhost:8080/capitulo20/api/equipo

```


## Referenciados y Embebidos


Entidad

Jugador


``
db.jugador.insertOne({"persona":{"nombre":"Shadia"},"equipo":{"equipo":"Barcelona A", "uid":"28fa37844-d4b2-4e31-ae6a-44c18b911d9b"}})
```


```
curl --location --request POST 'http://localhost:8080/capitulo20/api/jugador/' --header 'Content-Type: application/json' --data-raw '  "persona":{"nombre":"Shadia"},"equipo":{"equipo":"Barcelona A", "uid":"28fa37844-d4b2-4e31-ae6a-44c18b911d9b"}"
  
```

## Resumen

En esste capítulo se ha explicado mediante ejemplos como utilizar llaves primarias de tipo ObjectId y UUID.




