    # Capítulo 19



Cambiarlo por n






# Dynamic Repository

@DynamicRepository permite definir interface para repositorio en las cuales los metodos deben incluir un parametro de tipo DynamicInfo. 
Que contiene información sobre la base de datos y colección generados dinamicamente.

```java
public record DynamicInfo(String database, String collection) {

}


```



## ---

Este capítulo abarca el uso de  bases de datos y colecciones dinámicas con MongoDB.



Este capítulo incluye los temas:


* Bases Datos Dinámicas


* Colecciones Dinámicas

## Bases de datos dinámicas con jmoordbcore y MongoDB solo para embebido

La nueva versión de jmoordbcore, soporte especificar el nombre de la base de datos en tiempo de ejecución.

No aplica para entidades que utilizan referencias.


Agregue a su proyecto

```
 
 <properties>
   <version.jmoordbcore>2.0.2</version.jmoordbcore> 
 </properties>

<dependency>
  <groupId>com.github.avbravo</groupId>
  <artifactId>jmoordb-core</artifactId>
  <version>${version.jmoordbcore}</version>
</dependency>

<repositories>
 <repository>
    <id>jitpack.io</id>
    <url>https://jitpack.io</url>
  </repository>
</repositories>
```


Como recordaremos cuando se crea un proyecto que utilice Jmoordbcore se configura en el archivo microprofile-config.properties las propiedades de conexión y bases de datos.



```
mongodb.uri=mongodb://localhost:27017
#-- Database de configuración
mongodb.jmoordb= configurationjmoordbdb

#-- Database History
mongodb.databasehistory=historydb

#-- Database
mongodb.database=accreditation
mongodb.database1=sft
mongodb.database2=practicadb


```

Se define una entidad en el paquete model

```java
@Entity
public class Icono {
@Id(strategy = GenerationType.AUTO)
private Long idicono;
@Column
private String icono;

// set/get constructores
}

```


Se define un repositorio en el paquete repository

```java
@Repository(entity = Icono.class)
public interface IconoRepository extends CrudRepository<Icono, Long> {

    @Find
    public Optional<Icono> findByIcono(String icono);

    @Lookup
    public List<Icono> lookup(Search search);

    @Count()
    public Long count(Search... search);
    
    @CountLikeBy(caseSensitive = CaseSensitive.NO, likeByType = LikeByType.ANYWHERE)
    public Long countLikeByIcono(String icono);

    @LikeBy(caseSensitive = CaseSensitive.NO, typeOrder = TypeOrder.ASC, likeByType = LikeByType.ANYWHERE)
    public List<Icono> likeByIcono(String icono);

}

```

Creamos un endpoint mediante

```java
@Path("icono")
@Tag(name = "Información del icono", description = "End-point para entidad Icono")
@RolesAllowed({"admin"})
public class IconoController {

    
    // <editor-fold defaultstate="collapsed" desc="Inject">
    @Inject
    IconoRepository iconoRepository;
    
    @Inject
    HistoryRepository historyRepository;


// </editor-fold>


    // <editor-fold defaultstate="collapsed" desc="findAll">
    @GET
    @RolesAllowed({"admin"})
    @Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})
  
    public List<Icono> findAll() {
       
       return iconoRepository.findAll();
    }
// </editor-fold>

    // <editor-fold defaultstate="collapsed" desc="Icono findByIdicono">
    @GET
    @RolesAllowed({"admin"})
    @Path("{idicono}")
    public Icono findByIdicono(
            @Parameter(description = "El idicono", required = true, example = "1", schema = @Schema(type = SchemaType.NUMBER)) @PathParam("idicono") Long idicono) {



        return iconoRepository.findByPk(idicono).orElseThrow(
                () -> new WebApplicationException("No hay icono con idicono " + idicono, Response.Status.NOT_FOUND));

    }
// </editor-fold>
    // <editor-fold defaultstate="collapsed" desc="Icono findByIcononame">
    @GET
    @RolesAllowed({"admin"})
    @Path("icononame")
   
   
    public Icono findByIcono(@Parameter(description = "El icononame", required = true, example = "1", schema = @Schema(type = SchemaType.STRING)) @QueryParam("icono") final String icono) {



        return iconoRepository.findByIcono(icono).orElseThrow(
                () -> new WebApplicationException("No hay icono con icono " + icono, Response.Status.NOT_FOUND));

    }
//// </editor-fold>
// <editor-fold defaultstate="collapsed" desc="List<Icono> lookup(@QueryParam("filter") String filter, @QueryParam("sort") String sort, @QueryParam("page") Integer page, @QueryParam("size") Integer size)">

    @GET
    @Path("lookup")
    @RolesAllowed({"admin"})
   
    @Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})

    public List<Icono> lookup(@QueryParam("filter") String filter, @QueryParam("sort") String sort, @QueryParam("page") Integer page, @QueryParam("size") Integer size) {
        List<Icono> suggestions = new ArrayList<>();
        try {

        Search search = DocumentUtil.convertForLookup(filter, sort, page, size);
        suggestions = iconoRepository.lookup(search);

        } catch (Exception e) {
       
          MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + "error: " + e.getLocalizedMessage());
        }

        return suggestions;
    }

    // </editor-fold>

    // <editor-fold defaultstate="collapsed" desc="Response save">
    @POST
    @RolesAllowed({"admin"})
    public Response save(
            @RequestBody(description = "Crea un nuevo icono.", content = @Content(mediaType = "application/json", schema = @Schema(implementation = Icono.class))) Icono icono) {
iconoRepository.setDynamicDatabase("database1");
  Optional<Icono> iconoOptional=iconoRepository.save(icono);
        if(iconoOptional.isPresent()){
            saveHistory(icono);
               return Response.status(201).entity(iconoOptional.get()).build();
        }else{
              return Response.status(400).entity("Error " + iconoRepository.getJmoordbException().getLocalizedMessage()).build();
        }
    }
// </editor-fold>
    // <editor-fold defaultstate="collapsed" desc="Response update">

    @PUT
    @RolesAllowed({"admin"})
    public Response update(
            @RequestBody(description = "Crea un nuevo icono.", content = @Content(mediaType = "application/json", schema = @Schema(implementation = Icono.class))) Icono icono) {


         if(iconoRepository.update(icono)){
             saveHistory(icono);
               return Response.status(201).entity(icono).build();
        }else{
              return Response.status(400).entity("Error " + iconoRepository.getJmoordbException().getLocalizedMessage()).build();
        }
    }
// </editor-fold>

    // <editor-fold defaultstate="collapsed" desc="Response delete">
    @DELETE
    @RolesAllowed({"admin"})
    @Path("{idicono}")
    public Response delete(
            @Parameter(description = "El elemento idicono", required = true, example = "1", schema = @Schema(type = SchemaType.NUMBER)) @PathParam("idicono") Long idicono) {
        if(iconoRepository.deleteByPk(idicono) ==0L){
              return Response.status(201).entity(Boolean.TRUE).build();
        }else{
            return Response.status(400).entity("Error " + iconoRepository.getJmoordbException().getLocalizedMessage()).build();
        }
    }
    // </editor-fold>
    
     // <editor-fold defaultstate="collapsed" desc="Long count(@QueryParam("filter") String filter, @QueryParam("sort") String sort, @QueryParam("page") Integer page, @QueryParam("size") Integer size)">

    @GET
    @Path("count")
    @RolesAllowed({"admin"})
   

    @Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})

    public Long count(@QueryParam("filter") String filter, @QueryParam("sort") String sort, @QueryParam("page") Integer page, @QueryParam("size") Integer size) {
       Long result = 0L;
        try {

        Search search = DocumentUtil.convertForLookup(filter, sort, page, size);
        result = iconoRepository.count(search);

        } catch (Exception e) {
       
          MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + "error: " + e.getLocalizedMessage());
        }

        return result;
    }

    // </editor-fold>
     // <editor-fold defaultstate="collapsed" desc="private void saveHistory(Icono icono)">
    
    private void saveHistory(Icono icono){
        try {
                History history = new History.Builder()                 
               .collection("icono")
                    .idcollection(icono.getIdicono().toString())
                                .database("accreditation")
                    .data(icono.toString())
                    .actionHistory(icono.getActionHistory().get(icono.getActionHistory().size()-1)                  )
                     .build();
            historyRepository.save(history);
        } catch (Exception e) {
           ConsoleUtil.error("saveHistory() "+e.getLocalizedMessage());
        }
    }
     
    
// </editor-fold>
    
    
    
         // <editor-fold defaultstate="collapsed" desc="Long countLikeByRole(@QueryParam("role") String role)">
    @GET
    @Path("countlikebyicono")
    @Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})

    public Long countLikeByIcono(@QueryParam("icono") String icono) {
        Long result = 0L;
        try {

            
            result = iconoRepository.countLikeByIcono(icono);

        } catch (Exception e) {

            MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + "error: " + e.getLocalizedMessage());
        }

        return result;
    }

    // </editor-fold>
      // <editor-fold defaultstate="collapsed" desc="List<Role> likeByRole(@QueryParam("role") String role)">

    @GET
    @Path("likebyicono")
    @RolesAllowed({"admin"})
     @Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})

    public List<Icono> likeByIcono(@QueryParam("icono") String icono) {
        List<Icono> suggestions = new ArrayList<>();
        try {

       
        suggestions = iconoRepository.likeByIcono(icono);

        } catch (Exception e) {
       
          MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + "error: " + e.getLocalizedMessage());
        }

        return suggestions;
    }

    // </editor-fold>
}



```


Cuando se compila la aplicación se genera el código necesario para interactuar desde nuestra aplicación Java con MongoDB.


Cuando deseamos que el nombre de la base de datos no se tome del archivo microprofile-config.properties, podemos definirlo mediante el método .setDynamicDatabase("database"); del repositorio, que asignara el nombre de la base de datos que se pasa como parámetros y tendrá precedencia sobre el archivo de configuración. Tenga presente que se debe asignar a cada método para que sea funcional.

```java
    @POST
    @RolesAllowed({"admin"})
    public Response save(
            @RequestBody(description = "Crea un nuevo icono.", content = @Content(mediaType = "application/json", schema = @Schema(implementation = Icono.class))) Icono icono) {
        iconoRepository.setDynamicDatabase("database1");
        Optional<Icono> iconoOptional = iconoRepository.save(icono);
        if (iconoOptional.isPresent()) {
            saveHistory(icono);
            return Response.status(201).entity(iconoOptional.get()).build();
        } else {
            return Response.status(400).entity("Error " + iconoRepository.getJmoordbException().getLocalizedMessage()).build();
        }
    }
```

Método para buscar todos los documentos se asigna la base de datos.

```java
@GET
@RolesAllowed({"admin"})
@Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})
    
    public List<Icono> findAll() {
         iconoRepository.setDynamicDatabase("database1");
        return iconoRepository.findAll();
    }

```

Por el contrario si no se coloca se tomara la base de datos del archivo de configuración.


```java
@GET
@RolesAllowed({"admin"})
@Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})
    
    public List<Icono> findAll() {
       //  iconoRepository.setDynamicDatabase("database1");
        return iconoRepository.findAll();
    }

```



## Colecciones dinámicas con jmoordbcore y MongoDB solo para embebido

A partir de Jmoordbcore 1.3.0 es posible generar de manera dinámica colecciones en MongoDB.

No aplica para entidades que utilizan referencias.


## Sin especificar el nombre de base de datos o colección, se  toma la información del archivo microprofile-config.properties.

```

#-- Database
mongodb.database=accreditation
mongodb.database1=sft
mongodb.database2=practicadb

```


En el controller se realiza. 


```java

@Path("icono")
@RolesAllowed({"admin"})
public class IconoController {

    @Inject
    IconoRepository iconoRepository;


 @POST
 
 @RolesAllowed({"admin"})
 public Response save(
            @RequestBody(description = "Crea un nuevo icono.", content = @Content(mediaType = "application/json", schema = @Schema(implementation = Icono.class))) Icono icono) {
   Optional<Icono> iconoOptional = iconoRepository.save(icono);
        if (iconoOptional.isPresent()) {
            saveHistory(icono);
            return Response.status(201).entity(iconoOptional.get()).build();
        } else {
            return Response.status(400).entity("Error " + iconoRepository.getJmoordbException().getLocalizedMessage()).build();
        }
    }

}

```


## Agregando un nombre de base de datos y colección

Solo tiene que utilizar el método **setDynamicCollection()**.


```java

@Path("icono")
@RolesAllowed({"admin"})
public class IconoController {

   @Inject
   IconoRepository iconoRepository;


 @POST
 
 @RolesAllowed({"admin"})
 public Response save(
            @RequestBody(description = "Crea un nuevo icono.", content = @Content(mediaType = "application/json", schema = @Schema(implementation = Icono.class))) Icono icono) {
    iconoRepository.setDynamicDatabase("database2");
    iconoRepository.setDynamicCollection("icono_2");
    Optional<Icono> iconoOptional = iconoRepository.save(icono);
        if (iconoOptional.isPresent()) {
            saveHistory(icono);
            return Response.status(201).entity(iconoOptional.get()).build();
        } else {
            return Response.status(400).entity("Error " + iconoRepository.getJmoordbException().getLocalizedMessage()).build();
        }
    }

}

```



## Microservicios con colecciones dinámicas en jmoordbcore

Las bases de datos dinámicas y colecciones solo aplican para entidades que no contengan documentos referenciados.


![Image description](figura_19_00.png)

Para implementarlo se permite entidades que contengan o no documentos embebidos.


![Image description](figura_19_01.png)




En el Controller especifique la colección dinámica 

```java
  tarjetaRepository.setDynamicCollection(nameOfCollection + idproyecto);
```

Observe que el parámetro idproyecto lo recibe el endpoint y con el se genera la nueva colección.


```java

@Path("tarjeta")
@Tag(name = "Información del tarjeta", description = "End-point para entidad Tarjeta")
@RolesAllowed({"admin"})
@RequestScoped
public class TarjetaController implements Serializable{

    private String nameOfCollection = "tarjeta_";

    @Inject
    TarjetaRepository tarjetaRepository;
    @Inject
    HistoryRepository historyRepository;



    @GET
    @RolesAllowed({"admin"})
    @Path("idtarjetaidproyecto")
   
    public Tarjeta findByIdtarjetaIdproyecto(@QueryParam("idtarjeta") Long idtarjeta, @QueryParam("idproyecto") Long idproyecto) {
      
        tarjetaRepository.setDynamicCollection(nameOfCollection + idproyecto);
        return tarjetaRepository.findByPk(idtarjeta).orElseThrow(
                () -> new WebApplicationException("No hay tarjeta con idtarjeta " + idtarjeta, Response.Status.NOT_FOUND));

    }

    @POST
    @RolesAllowed({"admin"})
   
    public Response save(
            @RequestBody(description = "Crea un nuevo tarjeta.", content = @Content(mediaType = "application/json", schema = @Schema(implementation = Tarjeta.class))) Tarjeta tarjeta) {
     tarjetaRepository.setDynamicCollection(nameOfCollection + tarjeta.getIdproyecto());
        Optional<Tarjeta> tarjetaOptional = tarjetaRepository.save(tarjeta);
        if (tarjetaOptional.isPresent()) {
            saveHistory(tarjeta);

            return Response.status(201).entity(tarjetaOptional.get()).build();
        } else {
            return Response.status(400).entity("Error " + tarjetaRepository.getJmoordbException().getLocalizedMessage()).build();
        }

    }


    @PUT
    @RolesAllowed({"admin"})
    
    public Response update(
            @RequestBody(description = "Crea un nuevo tarjeta.", content = @Content(mediaType = "application/json", schema = @Schema(implementation = Tarjeta.class))) Tarjeta tarjeta) {
       tarjetaRepository.setDynamicCollection(nameOfCollection + tarjeta.getIdproyecto());
        if (tarjetaRepository.update(tarjeta)) {
            saveHistory(tarjeta);
            return Response.status(201).entity(tarjeta).build();
        } else {
            return Response.status(400).entity("Error " + tarjetaRepository.getJmoordbException().getLocalizedMessage()).build();
        }
    }


   
    @DELETE
    @RolesAllowed({"admin"})
    @Path("idtarjetaidproyecto")
    
    public Response delete(@QueryParam("idtarjeta") Long idtarjeta, @QueryParam("idproyecto") Long idproyecto) {
        tarjetaRepository.setDynamicCollection(nameOfCollection + idproyecto);
        if (tarjetaRepository.deleteByPk(idtarjeta) == 0L) {
            return Response.status(201).entity(Boolean.TRUE).build();
        } else {
            return Response.status(400).entity("Error " + tarjetaRepository.getJmoordbException().getLocalizedMessage()).build();
        }
    }

   
    @DELETE
    @Path("deletemany")
    @RolesAllowed({"admin"})
     public Response deleteMany(@QueryParam("filter") String filter, @QueryParam("idproyecto") Long idproyecto) {
        tarjetaRepository.setDynamicCollection(nameOfCollection + idproyecto);
        Search search = DocumentUtil.convertForLookup(filter, "", 0, 0);
        if (tarjetaRepository.deleteMany(search) == 0L) {
            return Response.status(201).entity(Boolean.TRUE).build();
        } else {
            return Response.status(400).entity("Error " + tarjetaRepository.getJmoordbException().getLocalizedMessage()).build();
        }
    }
    
    @GET
    @Path("lookup")
    @RolesAllowed({"admin"})
    
    public List<Tarjeta> lookup(@QueryParam("filter") String filter, @QueryParam("sort") String sort, @QueryParam("page") Integer page, @QueryParam("size") Integer size, @QueryParam("idproyecto") Long idproyecto) {
        List<Tarjeta> suggestions = new ArrayList<>();
        try {
            tarjetaRepository.setDynamicCollection(nameOfCollection + idproyecto);
            Search search = DocumentUtil.convertForLookup(filter, sort, page, size);

            suggestions = tarjetaRepository.lookup(search);

        } catch (Exception e) {

            MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + "error: " + e.getLocalizedMessage());
        }

        return suggestions;
    }

    @GET
    @Path("count")
    @RolesAllowed({"admin"})
    @Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})

    public Long count(@QueryParam("filter") String filter, @QueryParam("sort") String sort, @QueryParam("page") Integer page, @QueryParam("size") Integer size, @QueryParam("idproyecto") Long idproyecto) {
        Long result = 0L;
        try {
            tarjetaRepository.setDynamicCollection(nameOfCollection + idproyecto);
            Search search = DocumentUtil.convertForLookup(filter, sort, page, size);
            result = tarjetaRepository.count(search);

        } catch (Exception e) {

            MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + "error: " + e.getLocalizedMessage());
        }

        return result;
    }

  
  
    @GET
    @Path("countlikebytarjeta")
    @RolesAllowed({"admin"})
     @Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})

    public Long countLikeByTarjeta(@QueryParam("tarjeta") String tarjeta, @QueryParam("idproyecto") Long idproyecto) {
        Long result = 0L;
        try {
            tarjetaRepository.setDynamicCollection(nameOfCollection + idproyecto);
            result = tarjetaRepository.countLikeByTarjeta(tarjeta);

        } catch (Exception e) {

            MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + "error: " + e.getLocalizedMessage());
        }

        return result;
    }

  
    @GET
    @Path("countlikebydescripcion")
    @RolesAllowed({"admin"})
     @Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})

    public Long countLikeByDescripcion(@QueryParam("descripcion") String descripcion, @QueryParam("idproyecto") Long idproyecto) {
        Long result = 0L;
        try {
            tarjetaRepository.setDynamicCollection(nameOfCollection + idproyecto);
            result = tarjetaRepository.countLikeByDescripcion(descripcion);

        } catch (Exception e) {

            MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + "error: " + e.getLocalizedMessage());
        }

        return result;
    }

   
    @GET
    @Path("likebytarjeta")
    @RolesAllowed({"admin"})
    @Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})

    public List<Tarjeta> likeByName(@QueryParam("tarjeta") String tarjeta, @QueryParam("idproyecto") Long idproyecto) {
        List<Tarjeta> suggestions = new ArrayList<>();
        try {
            tarjetaRepository.setDynamicCollection(nameOfCollection + idproyecto);
            suggestions = tarjetaRepository.likeByTarjeta(tarjeta);

        } catch (Exception e) {

            MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + "error: " + e.getLocalizedMessage());
        }

        return suggestions;
    }

   
    @GET
    @Path("likebytarjetasearch")
    @RolesAllowed({"admin"})
    @Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})
    public List<Tarjeta> searchLikeByTarjeta(@QueryParam("tarjeta") String tarjeta, @QueryParam("filter") String filter, @QueryParam("sort") String sort, @QueryParam("page") Integer page, @QueryParam("size") Integer size, @QueryParam("idproyecto") Long idproyecto) {
        List<Tarjeta> suggestions = new ArrayList<>();
        try {
            tarjetaRepository.setDynamicCollection(nameOfCollection + idproyecto);
            Search search = DocumentUtil.convertForLookup(filter, sort, page, size);
            suggestions = tarjetaRepository.searchLikeByTarjeta(tarjeta, search);
        } catch (Exception e) {
            MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + "error: " + e.getLocalizedMessage());
        }
        return suggestions;
    }

    
    @GET
    @Path("likebydescripcionsearch")
    @RolesAllowed({"admin"})
    @Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})
    public List<Tarjeta> searchLikeByDescripcion(@QueryParam("descripcion") String descripcion, @QueryParam("filter") String filter, @QueryParam("sort") String sort, @QueryParam("page") Integer page, @QueryParam("size") Integer size, @QueryParam("idproyecto") Long idproyecto) {
        List<Tarjeta> suggestions = new ArrayList<>();
        try {
            tarjetaRepository.setDynamicCollection(nameOfCollection + idproyecto);
            Search search = DocumentUtil.convertForLookup(filter, sort, page, size);
            suggestions = tarjetaRepository.searchLikeByDescripcion(descripcion, search);
        } catch (Exception e) {
            MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + "error: " + e.getLocalizedMessage());
        }
        return suggestions;
    }

   
   
   
}

```


El repositorio tendría una estructura similar a 

```java
@Repository(database = "{mongodb.database1}", entity = Tarjeta.class)
public interface TarjetaRepository extends CrudRepository<Tarjeta, Long> {

    @Lookup
    public List<Tarjeta> lookup(Search search);

    @Count()
    public Long count(Search... search);
    
    @CountLikeBy(caseSensitive = CaseSensitive.NO, likeByType = LikeByType.ANYWHERE)
    public Long countLikeByTarjeta(String tarjeta);
       
    @CountLikeBy(caseSensitive = CaseSensitive.NO, likeByType = LikeByType.ANYWHERE)
    public Long countLikeByDescripcion(String descripcion);

    @SearchCountLikeBy(caseSensitive = CaseSensitive.NO, likeByType = LikeByType.ANYWHERE)
    public Long searchCountLikeByTarjeta(String tarjeta, Search search);
    
    @SearchCountLikeBy(caseSensitive = CaseSensitive.NO, likeByType = LikeByType.ANYWHERE)
    public Long searchCountLikeByDescripcion(String descripcion, Search search);
    
    @LikeBy(caseSensitive = CaseSensitive.NO, typeOrder = TypeOrder.ASC, likeByType = LikeByType.ANYWHERE)
    public List<Tarjeta> likeByTarjeta(String tarjeta);
    
    @LikeBy(caseSensitive = CaseSensitive.NO, typeOrder = TypeOrder.ASC, likeByType = LikeByType.ANYWHERE)
    public List<Tarjeta> likeByTarjetaPagination(String tarjeta, Pagination pagination);
    
   
    
    @LikeBy(caseSensitive = CaseSensitive.NO, typeOrder = TypeOrder.ASC, likeByType = LikeByType.ANYWHERE)
    public List<Tarjeta> likeByDescripcion(String descripcion);


   @SearchLikeBy(caseSensitive = CaseSensitive.NO, typeOrder = TypeOrder.ASC, likeByType = LikeByType.ANYWHERE)
    public List<Tarjeta> searchLikeByTarjeta(String tarjeta, Search search);

    @SearchLikeBy(caseSensitive = CaseSensitive.NO, typeOrder = TypeOrder.ASC, likeByType = LikeByType.ANYWHERE)
    public List<Tarjeta> searchLikeByDescripcion(String descripcion, Search search);

    @Find
    public List<Tarjeta> findByFechainicialGreaterThanEqualAndFechafinalLessThanEqual(@ExcludeTime Date start, @ExcludeTime Date end);
}

```


---

## En el cliente utilizamos Microprofile Rest-Client

En el archivo microprofile-config.properties

```
com.sft.restclient.TarjetaRestClient/mp-rest/url=http://localhost:9002/accreditation/api/
```


### Crear el RestClient

```java

 @RegisterRestClient()
@Path("/tarjeta")

//@ApplicationScoped
public interface TarjetaRestClient {

@GET
    @Path("idtarjetaidproyecto")
    public Tarjeta findByIdtarjetaproyecto(@QueryParam("idtarjeta") Long idtarjeta, @QueryParam("idproyecto") Long idproyecto);

 @GET
    @Path("tarjeta")
    public List<Tarjeta> findByTarjeta(@Parameter(description = "El tarjeta", required = true, example = "1", schema = @Schema(type = SchemaType.STRING)) @QueryParam("tarjeta") final String tarjeta,@QueryParam("idproyecto") Long idproyecto);
//// </editor-fold>

    

    // <editor-fold defaultstate="collapsed" desc="Response save">
    @POST

    public Response save(@RequestBody(description = "Crea un nuevo tarjeta.", content = @Content(mediaType = "application/json", schema = @Schema(implementation = Tarjeta.class))) Tarjeta tarjeta);
// </editor-fold>
    // <editor-fold defaultstate="collapsed" desc="Response update">

    @PUT

    public Response update(@RequestBody(description = "Actualiza la tarjeta.", content = @Content(mediaType = "application/json", schema = @Schema(implementation = Tarjeta.class))) Tarjeta tarjeta);

// </editor-fold>
    // <editor-fold defaultstate="collapsed" desc="Response delete">
    @DELETE
//    @Path("{idtarjeta}")
    @Path("idtarjetaidproyecto")
//    public Response delete(@Parameter(description = "El elemento idtarjeta", required = true, example = "1", schema = @Schema(type = SchemaType.NUMBER)) @PathParam("idtarjeta") Long idtarjeta);
    public Response delete(@QueryParam("idtarjeta") Long idtarjeta, @QueryParam("idproyecto") Long idproyecto);
    // </editor-fold>
    
    
    // <editor-fold defaultstate="collapsed" desc="Response deleteMany(@QueryParam("filter") String filter ,@QueryParam("idproyecto") Long idproyecto)">
     @DELETE
     @Path("deletemany")
public Response deleteMany(@QueryParam("filter") String filter ,@QueryParam("idproyecto") Long idproyecto ) ;
  // </editor-fold>
    
    // <editor-fold defaultstate="collapsed" desc="List<Tarjeta> lookup(@QueryParam("filter") String filter, @QueryParam("sort") String sort,  @QueryParam("page") Integer page, @QueryParam("size") Integer size,@QueryParam("idproyecto") Long idproyecto)">
    @GET
    @Path("lookup")
    public List<Tarjeta> lookup(@QueryParam("filter") String filter, @QueryParam("sort") String sort, @QueryParam("page") Integer page, @QueryParam("size") Integer size,@QueryParam("idproyecto") Long idproyecto);
    // </editor-fold>    
    
    
 // <editor-fold defaultstate="collapsed" desc="public Long count(@QueryParam("filter") String filter, @QueryParam("sort") String sort, @QueryParam("page") Integer page, @QueryParam("size") Integer size,@QueryParam("idproyecto") Long idproyecto);">
    @GET
    @Path("count")
    public Long count(@QueryParam("filter") String filter, @QueryParam("sort") String sort, @QueryParam("page") Integer page, @QueryParam("size") Integer size,@QueryParam("idproyecto") Long idproyecto);
    // </editor-fold>    

    // <editor-fold defaultstate="collapsed" desc="Long countLikeByTarjeta(@QueryParam("tarjeta") String tarjeta,@QueryParam("idproyecto") Long idproyecto)">
    @GET
   @Path("countlikebytarjeta")
     public Long countLikeByTarjeta(@QueryParam("tarjeta") String tarjeta,@QueryParam("idproyecto") Long idproyecto);
    // </editor-fold>    
 
    // <editor-fold defaultstate="collapsed" desc="Long countLikeByDescripcion(@QueryParam("descripcion") String descripcion,@QueryParam("idproyecto") Long idproyecto)">
    @GET
   @Path("countlikebydescripcion")
     public Long countLikeByDescripcion(@QueryParam("descripcion") String descripcion,@QueryParam("idproyecto") Long idproyecto);
    // </editor-fold>    
 

// <editor-fold defaultstate="collapsed" desc="public List<Tarjeta> likeByTarjeta(@QueryParam("tarjeta") String tarjeta,@QueryParam("idproyecto") Long idproyecto)">

    @GET
    @Path("likebytarjeta")
    public List<Tarjeta> likeByTarjeta(@QueryParam("tarjeta") String tarjeta,@QueryParam("idproyecto") Long idproyecto);
    // </editor-fold>
// <editor-fold defaultstate="collapsed" desc="public List<Tarjeta> likeByTarjetaPagination(@QueryParam("tarjeta") String tarjeta, @QueryParam("pagination") Pagination pagination,@QueryParam("idproyecto") Long idproyecto)">

    @GET
    @Path("likebytarjetapagination")
    public List<Tarjeta> likeByTarjetaPagination(@QueryParam("tarjeta") String tarjeta, @QueryParam("page") Integer page, @QueryParam("size") Integer siz,@QueryParam("idproyecto") Long idproyectoe);
    // </editor-fold>


// <editor-fold defaultstate="collapsed" desc="List<Tarjeta> likeByDescripcion(@QueryParam("descripcion") String descripcion,@QueryParam("idproyecto") Long idproyecto)">

    @GET
    @Path("likebydescripcion")
    public List<Tarjeta> likeByDescripcion(@QueryParam("descripcion") String descripcion,@QueryParam("idproyecto") Long idproyecto);
    // </editor-fold>
    
    // <editor-fold defaultstate="collapsed" desc="List<Tarjeta> betweenDate(@QueryParam("fechainicial") Date fechainicial, @QueryParam("fechafinal") Date fechafinal,@QueryParam("idproyecto") Long idproyecto)">
      @GET
    @Path("betweendate")
    public List<Tarjeta> betweenDate(@QueryParam("fechainicial") Date fechainicial, @QueryParam("fechafinal") Date fechafinal,@QueryParam("idproyecto") Long idproyecto) ;
    // </editor-fold>
    
    
    
    @GET
    @Path("likebytarjetasearch")
    @Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})
    List<Tarjeta> searchLikeByTarjeta(@QueryParam("tarjeta") String tarjeta, @QueryParam("filter") String filter, @QueryParam("sort") String sort, @QueryParam("page") Integer page, @QueryParam("size") Integer size,@QueryParam("idproyecto") Long idproyecto);
    
   
    @GET
    @Path("searchcountlikebytarjeta")
    @Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})
   public  Long searchCountLikeByTarjeta(@QueryParam("tarjeta") String tarjeta, @QueryParam("filter") String filter, @QueryParam("sort") String sort, @QueryParam("page") Integer page, @QueryParam("size") Integer size,@QueryParam("idproyecto") Long idproyecto);

   
    @GET
    @Path("likebydescripcionsearch")
    @Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})
    List<Tarjeta> searchLikeByDescripcion(@QueryParam("descripcion") String descripcion, @QueryParam("filter") String filter, @QueryParam("sort") String sort, @QueryParam("page") Integer page, @QueryParam("size") Integer size,@QueryParam("idproyecto") Long idproyecto);
    
    
    @GET
    @Path("searchcountlikebydescripcion")
    @Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})
   public  Long searchCountLikeByDescripcion(@QueryParam("descripcion") String descripcion, @QueryParam("filter") String filter, @QueryParam("sort") String sort, @QueryParam("page") Integer page, @QueryParam("size") Integer size,@QueryParam("idproyecto") Long idproyecto);
}

```


### Crear TarjetaServices

```java
public interface TarjetaServices {
//     public List<Tarjeta> findAll();
    public Optional<Tarjeta> save(Tarjeta tarjeta);

    public Boolean update(Tarjeta tarjeta);

    public Boolean deleteMany(Bson filter,Long idproyecto);

//    public Optional<Tarjeta> findByIdtarjeta(Long idtarjeta);
    public Optional<Tarjeta> findByIdtarjetaIdproyecto(Long idtarjeta, Long idproyecto);

    public List<Tarjeta> lookup(Bson filter, Document sort, Integer page, Integer size, Long idproyecto);

    public Boolean tarjetaExistInSprint(String tarjetaName, Long idproyecto, Long idsprint);

    public Boolean tarjetaExistInBacklog(String tarjetaName, Long idproyecto, Long idsprint);

    public Optional<Tarjeta> tarjetaConIgualNombreInSprint(String tarjetaName, Long idproyecto, Long idsprint);

    public Long count(Bson filter, Document sort, Integer page, Integer size, Long idproyecto);

    public Long totalPorColumna(Proyecto proyecto, String columna, Boolean storeInBacklog);

    public List<Comentario> ordenarComentarioPorFechaDescendente(Tarjeta tarjeta);

    public List<Tarea> ordenarTareaPorCompletadoDescendente(Tarjeta tarjeta);
    public List<Tarea> ordenarTareaPorOrden(Tarjeta tarjeta);

    public List<Impedimento> ordenarImpedimentoDescendente(Tarjeta tarjeta);

    public Boolean isMiembroAutorizedInTarjetaForanea(Tarjeta tarjeta, User userLogged, UserView userViewForaneo);

    public Boolean equalsExcludedNameOfTarjeta(Tarjeta tarjeta, Tarjeta other);

    public Boolean isEstimacionValida(Tarjeta tarjeta);

    public String colorTarjeta(Tarjeta tarjeta);

    public Long countLikeByTarjeta(String tarjeta, Long idproyecto);

    public Long countLikeByDescripcion(String descripcion, Long idproyecto);

    public List<Tarjeta> likeByTarjeta(String tarjeta, Long idproyecto);

    public List<Tarjeta> likeByTarjetaPagination(String tarjeta, Pagination pagination, Long idproyecto);

    public List<Tarjeta> likeByDescripcion(String descripcion, Long idproyecto);

    public List<Tarjeta> betweenDate(@QueryParam("fechainicial") Date fechainicial, @QueryParam("fechafinal") Date fechafinal, Long idproyecto);

    public List<Tarjeta> searchLikeByTarjeta(String tarjeta, Bson filter, Document sort, Integer page, Integer size, Long idproyecto);
    public Long searchCountLikeByTarjeta(String tarjeta, Bson filter, Document sort, Integer page, Integer size, Long idproyecto);
    
    public List<Tarjeta> searchLikeByDescripcion(String descripcion, Bson filter, Document sort, Integer page, Integer size, Long idproyecto);
        public Long searchCountLikeByDescripcion(String descripcion, Bson filter, Document sort, Integer page, Integer size, Long idproyecto);
    

    public List<Tarjeta> validarFechaFinalEsteSprintActual(List<Tarjeta> list, Sprint sprint);
    public List<Tarjeta> validarFechaFinalSprintPlanTrabajo(List<Tarjeta> list, Sprint sprint);
    
    public List<Tarjeta> orderListForIdTarjetaReserve(List<Tarjeta> list);
    
    public Integer positionOfTarjeta(Tarjeta tarjeta, List<Tarjeta> tarjetas);
    
    public TotalesTarjetasEstadistica calcularTotalesTarjetasEstadistica(Long iduser, Long idproyecto);
    
            
   
}

```

### Crear la implementación

```java
@ApplicationScoped
public class TarjetaServicesImpl implements TarjetaServices {
// <editor-fold defaultstate="collapsed" desc="@Inject">

    @Inject
    JmoordbResourcesFiles rf;
    // </editor-fold>
    // <editor-fold defaultstate="collapsed" desc="Microprofile Rest Client">
    @Inject
    TarjetaRestClient tarjetaRestClient;
    @Inject
    TarjetaServices tarjetaServices;
// </editor-fold>

//    // <editor-fold defaultstate="collapsed" desc="List<Tarjeta> findAll()">
//    @Override
//    public List<Tarjeta> findAll() {
//        return tarjetaRestClient.findAll();
//    }
//// </editor-fold>
    // <editor-fold defaultstate="collapsed" desc="Boolean update(Tarjeta tarjeta)">

    @Override
    public Boolean update(Tarjeta tarjeta) {
        Boolean result = Boolean.FALSE;
        try {

            Integer status = tarjetaRestClient.update(tarjeta).getStatus();

            if (status == 201) {
                result = Boolean.TRUE;
            }

        } catch (Exception e) {
            FacesUtil.errorMessage(FacesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
        }
        return result;
    }
// </editor-fold>

//    // <editor-fold defaultstate="collapsed" desc="Optional<Tarjeta> findByIdtarjeta(Long idtarjeta)">
//    @Override
//    public Optional<Tarjeta> findByIdtarjeta(Long idtarjeta) {
//
//        try {
//            Tarjeta result = tarjetaRestClient.findByIdtarjeta(idtarjeta);
//            if (result == null || result.getIdtarjeta() == null) {
//
//            } else {
//                return Optional.of(result);
//            }
//        } catch (Exception e) {
//            FacesUtil.errorMessage(FacesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
//        }
//        return Optional.empty();
//
//    }
//    // </editor-fold>
    // <editor-fold defaultstate="collapsed" desc="Optional<Tarjeta> findByIdtarjetaIdproyecto(Long idtarjeta, Long idproyecto)">
    @Override
    public Optional<Tarjeta> findByIdtarjetaIdproyecto(Long idtarjeta, Long idproyecto) {

        try {
            ConsoleUtil.test("\t >>>>>"+FacesUtil.nameOfClassAndMethod() + " <<<<<");
            ConsoleUtil.test("\t >>> idtarjeta "+idtarjeta + " idproyecto "+idproyecto);
            Tarjeta result = tarjetaRestClient.findByIdtarjetaproyecto(idtarjeta,idproyecto);
            ConsoleUtil.test("\t llego a 2");
            if (result == null || result.getIdtarjeta() == null) {

            } else {
                return Optional.of(result);
            }
        } catch (Exception e) {
            FacesUtil.errorMessage(FacesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
        }
        return Optional.empty();

    }
    // </editor-fold>
    // <editor-fold defaultstate="collapsed" desc="List<Tarjeta> lookup(Bson filter, Document sort, Integer page, Integer size, Long idproyecto)">

    @Override
    public List<Tarjeta> lookup(Bson filter, Document sort, Integer page, Integer size, Long idproyecto) {
        List<Tarjeta> tarjetaList = new ArrayList<>();
        try {

            tarjetaList = tarjetaRestClient.lookup(
                    EncodeUtil.encodeBson(filter),
                    EncodeUtil.encodeBson(sort),
                    page, size, idproyecto);

        } catch (Exception e) {

            FacesUtil.errorMessage(FacesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
        }

        return tarjetaList;
    }
// </editor-fold>
    // <editor-fold defaultstate="collapsed" desc="Boolean deleteMany(Bson filter, Long idproyecto) ">

    @Override
    public Boolean deleteMany(Bson filter, Long idproyecto) {
        Boolean result = Boolean.FALSE;
        try {
            Integer status = tarjetaRestClient.deleteMany(EncodeUtil.encodeBson(filter),idproyecto).getStatus();
            if (status == 201) {
                result = Boolean.TRUE;
            }
        } catch (Exception e) {
            FacesUtil.errorMessage(FacesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
        }
        return result;
    }
// </editor-fold>

    // <editor-fold defaultstate="collapsed" desc="Long count(Bson filter, Document sort, Integer page, Integer size, Long idproyecto)">
    @Override
    public Long count(Bson filter, Document sort, Integer page, Integer size, Long idproyecto) {
        Long result = 0L;
        try {
            result = tarjetaRestClient.count(
                    EncodeUtil.encodeBson(filter),
                    EncodeUtil.encodeBson(sort),
                    page, size,idproyecto);
        } catch (Exception e) {
            FacesUtil.errorMessage(FacesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
        }
        return result;
    }
    // </editor-fold>

    // <editor-fold defaultstate="collapsed" desc="Optional<Tarjeta>(Tarjeta tarjeta)">
    @Override
    public Optional<Tarjeta> save(Tarjeta tarjeta) {
        try {

            Response response = tarjetaRestClient.save(tarjeta);

            if (response.getStatus() == 400) {

                String error = (response.readEntity(String.class));

                return Optional.empty();
            }

            Tarjeta result = (Tarjeta) (response.readEntity(Tarjeta.class));

            return Optional.of(result);

        } catch (Exception e) {
            FacesUtil.errorMessage(FacesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
        }
        return Optional.empty();
    }
// </editor-fold>

    // <editor-fold defaultstate="collapsed" desc="Boolean tarjetaExist(String tarjetaName, Long idproyecto, Long idsprint)">
    @Override
    public Boolean tarjetaExistInSprint(String tarjetaName, Long idproyecto, Long idsprint) {
        Boolean result = Boolean.FALSE;
        try {
            Integer page = 0;
            Integer size = 0;
            Bson filter = and(
                    eq("idproyecto", idproyecto),
                    eq("idsprint", idsprint),
                    eq("active", true),
                    eq("tarjeta", tarjetaName)
            );

            Document sort = new Document("idtarjeta", 1);
            if (count(filter, sort, page, size,idproyecto) > 0) {
                result = Boolean.TRUE;
            }

        } catch (Exception e) {
            FacesUtil.errorMessage(FacesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
        }
        return result;
    }

    // </editor-fold>
    // <editor-fold defaultstate="collapsed" desc="Boolean tarjetaExist(Bson filter, Document sort, Integer page, Integer size)">
    @Override
    public Boolean tarjetaExistInBacklog(String tarjetaName, Long idproyecto, Long idsprint) {
        Boolean result = Boolean.FALSE;
        try {
            Integer page = 0;
            Integer size = 0;
            Bson filter = and(
                    eq("idproyecto", idproyecto),
                    eq("idsprint", 0L),
                    eq("active", true),
                    eq("tarjeta", tarjetaName)
            );

            Document sort = new Document("idtarjeta", 1);
            if (count(filter, sort, page, size,idproyecto) > 0) {
                result = Boolean.TRUE;
            }

        } catch (Exception e) {
            FacesUtil.errorMessage(FacesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
        }
        return result;
    }

    // </editor-fold>
    // <editor-fold defaultstate="collapsed" desc=" List<Tarjeta> tarjetaConIgualNombreInSprint(String tarjetaName, Long idproyecto, Long idsprint)">
    @Override
    public Optional<Tarjeta> tarjetaConIgualNombreInSprint(String tarjetaName, Long idproyecto, Long idsprint) {
        Optional<Tarjeta> result = Optional.empty();
        try {
            Integer page = 0;
            Integer size = 0;
            Bson filter = and(
                    eq("idproyecto", idproyecto),
                    eq("idsprint", idsprint),
                    eq("active", true),
                    eq("tarjeta", tarjetaName)
            );

            Document sort = new Document("idtarjeta", 1);
            List<Tarjeta> list = lookup(filter, sort, page, size,idproyecto);
            if (list == null || list.isEmpty()) {
                return Optional.empty();
            }
            result = Optional.of(list.get(0));
        } catch (Exception e) {
            FacesUtil.errorMessage(FacesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
        }
        return result;
    }

    // </editor-fold>
    // <editor-fold defaultstate="collapsed" desc="List<Comentario> ordenarComentario(Tarjeta tarjeta) ">
    /**
     * Ordena los comentarios por fecha
     *
     * @param tarjeta
     * @return
     */
    @Override
    public List<Comentario> ordenarComentarioPorFechaDescendente(Tarjeta tarjeta) {
        List<Comentario> result = new ArrayList<>();
        try {

            if (tarjeta.getComentario() == null || tarjeta.getComentario().isEmpty()) {
                return result;
            }
            Comparator<Comentario> comparator
                    = (c1, c2) -> c1.getFecha().compareTo(c2.getFecha());

            tarjeta.getComentario().sort(comparator.reversed());

        } catch (Exception e) {
//            FacesUtil.errorMessage(FacesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
        }
        return tarjeta.getComentario();
    }

    // </editor-fold>
    // <editor-fold defaultstate="collapsed" desc="Boolean isMiembroAutorizedInTarjetaForanea(Tarjeta tarjeta, User userLogged, UserView userViewForaneo)">
    @Override
    public Boolean isMiembroAutorizedInTarjetaForanea(Tarjeta tarjeta, User userLogged, UserView userViewForaneo) {
        Boolean result = Boolean.FALSE;
        try {
            if (tarjeta.getForeaneo()) {
                if (userViewForaneo.getIduser().equals(userLogged.getIduser())) {

                } else {
                    result = Boolean.TRUE;
                }
            } else {
                result = Boolean.TRUE;
            }
        } catch (Exception e) {
//            FacesUtil.errorMessage(FacesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
        }
        return result;
    }
    // </editor-fold>

    // <editor-fold defaultstate="collapsed" desc="List<Comentario> ordenarTareaPorCompletadoDescendente(Tarjeta tarjeta) ">
    @Override
    public List<Tarea> ordenarTareaPorCompletadoDescendente(Tarjeta tarjeta) {
        List<Tarea> result = new ArrayList<>();
        try {

            if (tarjeta.getTarea() == null || tarjeta.getTarea().isEmpty()) {
                return result;
            }
            Comparator<Tarea> comparator
                    = (c1, c2) -> c1.getCompletado().compareTo(c2.getCompletado());

            tarjeta.getTarea().sort(comparator);
//           tarjeta.getTarea().sort(comparator.reversed());

        } catch (Exception e) {
//            FacesUtil.errorMessage(FacesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
        }
        return tarjeta.getTarea();
    }

    // </editor-fold>
    // <editor-fold defaultstate="collapsed" desc="List<Tarea> ordenarTareaPorOrden(Tarjeta tarjeta)">
    @Override
    public List<Tarea> ordenarTareaPorOrden(Tarjeta tarjeta) {
        List<Tarea> result = new ArrayList<>();
        try {
            if (tarjeta.getTarea() == null || tarjeta.getTarea().isEmpty()) {
                return result;
            }
            Comparator<Tarea> comparator
                    = (c1, c2) -> c1.getOrden().compareTo(c2.getOrden());

            tarjeta.getTarea().sort(comparator);

        } catch (Exception e) {
//            FacesUtil.errorMessage(FacesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
        }
        return tarjeta.getTarea();
    }
    // </editor-fold>

    // <editor-fold defaultstate="collapsed" desc="List<Impedimento> ordenarImpedimentoDescendente(Tarjeta tarjeta) ">
    @Override
    public List<Impedimento> ordenarImpedimentoDescendente(Tarjeta tarjeta) {
        List<Impedimento> result = new ArrayList<>();
        try {

            if (tarjeta.getImpedimento() == null || tarjeta.getImpedimento().isEmpty()) {
                return result;
            }
            Comparator<Impedimento> comparator
                    = (c1, c2) -> c1.getImpedimento().compareTo(c2.getImpedimento());

            tarjeta.getImpedimento().sort(comparator.reversed());

        } catch (Exception e) {
//            FacesUtil.errorMessage(FacesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
        }
        return tarjeta.getImpedimento();
    }
    // </editor-fold>

    // <editor-fold defaultstate="collapsed" desc="Long totalPorColumna(Proyecto proyecto, String columna,Boolean storeInBacklog) ">
    /**
     * *
     *
     * @param proyecto
     * @param columna
     * @param storeInBacklog
     * @return devuelve el total de columnas
     */
    @Override
    public Long totalPorColumna(Proyecto proyecto, String columna, Boolean storeInBacklog) {
        Long result = 0L;

        try {
            Integer page = 0;
            Integer size = 0;
            Document sortTarjeta = new Document("idtarjeta", 1);
            /**
             * CargarTarjetas
             */

            if (!storeInBacklog) {
                Bson filter0 = eq("idproyecto", proyecto.getIdproyecto());
                Bson filter = and(filter0, eq("active", Boolean.TRUE),
                        eq("columna", columna));
                result = count(filter, sortTarjeta, page, size,proyecto.getIdproyecto());
            } else {
                Bson filter0 = eq("idproyecto", proyecto.getIdproyecto());
                Bson filter = and(filter0, eq("active", Boolean.TRUE),
                        eq("columna", columna),
                        eq("backlog", storeInBacklog));
                result = count(filter, sortTarjeta, page, size,proyecto.getIdproyecto());
            }

        } catch (Exception e) {
            FacesUtil.errorMessage(FacesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
        }
        return result;
    }
    // </editor-fold>

    // <editor-fold defaultstate="collapsed" desc="Boolean equalsExcludedNameOfTarjeta(Tarjeta tarjeta, Tarjeta other) ">
    @Override
    public Boolean equalsExcludedNameOfTarjeta(Tarjeta tarjeta, Tarjeta other) {
        Boolean result = Boolean.FALSE;
        try {
            if (tarjeta == null) {
                return false;
            }

            if (!Objects.equals(tarjeta.getDescripcion(), other.getDescripcion())) {
                return false;
            }
            if (!Objects.equals(tarjeta.getPrioridad(), other.getPrioridad())) {
                return false;
            }
            if (!Objects.equals(tarjeta.getEstimacion(), other.getEstimacion())) {
                return false;
            }
            if (!Objects.equals(tarjeta.getColumna(), other.getColumna())) {
                return false;
            }
            if (!Objects.equals(tarjeta.getIdtarjeta(), other.getIdtarjeta())) {
                return false;
            }
            if (!Objects.equals(tarjeta.getUserView(), other.getUserView())) {
                return false;
            }
            if (!Objects.equals(tarjeta.getFechainicial(), other.getFechainicial())) {
                return false;
            }
            if (!Objects.equals(tarjeta.getFechafinal(), other.getFechafinal())) {
                return false;
            }
            if (!Objects.equals(tarjeta.getIcono(), other.getIcono())) {
                return false;
            }
            if (!Objects.equals(tarjeta.getTipotarjeta(), other.getTipotarjeta())) {
                return false;
            }
            if (!Objects.equals(tarjeta.getIdsprint(), other.getIdsprint())) {
                return false;
            }
            if (!Objects.equals(tarjeta.getIdproyecto(), other.getIdproyecto())) {
                return false;
            }
            if (!Objects.equals(tarjeta.getBacklog(), other.getBacklog())) {
                return false;
            }
            if (!Objects.equals(tarjeta.getActive(), other.getActive())) {
                return false;
            }
            if (!Objects.equals(tarjeta.getTarea(), other.getTarea())) {
                return false;
            }
            if (!Objects.equals(tarjeta.getComentario(), other.getComentario())) {
                return false;
            }
            if (!Objects.equals(tarjeta.getEtiqueta(), other.getEtiqueta())) {
                return false;
            }
            if (!Objects.equals(tarjeta.getArchivo(), other.getArchivo())) {
                return false;
            }
            if (!Objects.equals(tarjeta.getImpedimento(), other.getImpedimento())) {
                return false;
            }
            if (!Objects.equals(tarjeta.getForeaneo(), other.getForeaneo())) {
                return false;
            }
            if (!Objects.equals(tarjeta.getActionHistory(), other.getActionHistory())) {
                return false;
            }
            result = Boolean.TRUE;
        } catch (Exception e) {
            FacesUtil.errorMessage(FacesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
        }
        return result;
    }
    // </editor-fold>

    // <editor-fold defaultstate="collapsed" desc="Boolean isEstimacionValida(Tarjeta tarjeta)">
    @Override
    public Boolean isEstimacionValida(Tarjeta tarjeta) {
        Boolean result = Boolean.TRUE;
        try {

            if (tarjeta.getEstimacion() == null || tarjeta.getEstimacion().equals("")) {
                FacesUtil.warningDialog(rf.fromCore("warning.warning"), rf.fromMessage("warning.ingreseestimacionformato"));
                return Boolean.FALSE;
            }

            if (tarjeta.getEstimacion().indexOf(":") == -1) {
                FacesUtil.warningDialog(rf.fromCore("warning.warning"), rf.fromMessage("warning.ingreseestimacionformato"));
                return Boolean.FALSE;
            }

            if (tarjeta.getEstimacion().indexOf("-") != -1) {
                FacesUtil.warningDialog(rf.fromCore("warning.warning"), rf.fromMessage("warning.ingreseestimacionformato"));
                return Boolean.FALSE;
            }

        } catch (Exception e) {
            FacesUtil.errorMessage(FacesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
        }
        return result;
    }
    // </editor-fold>

    // <editor-fold defaultstate="collapsed" desc="String colorTarjeta(Tarjeta tarjeta)">
    /**
     * Listado de colores https://tailwindcss.com/docs/background-color
     *
     * @param tarjeta
     * @return
     */
    @Override
    public String colorTarjeta(Tarjeta tarjeta) {
        String color = "";
        try {
            switch (tarjeta.getPrioridad().toLowerCase()) {

                case "urgente":
                    color = "bg-orange-900";
                    break;

                case "alta":
                    color = "bg-purple-900";
                    break;
                case "baja":
////color ="bg-green-950";
//color ="bg-teal-600";
////color ="bg-green-600";
//                    color = "bg-indigo-900";
//                    color = "bg-indigo-700";
                    color = "bg-teal-900";
                    break;

                case "media":
                    color = "";
                    break;
            }
        } catch (Exception e) {
            FacesUtil.errorMessage(FacesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
        }
        return color;
    }
// </editor-fold>

    // <editor-fold defaultstate="collapsed" desc="public Long countLikeByTarjeta(@QueryParam("tarjeta") String tarjeta, Long idproyecto)">
    @Override
    public Long countLikeByTarjeta(String tarjeta, Long idproyecto) {
        Long result = 0L;
        try {
            result = tarjetaRestClient.countLikeByTarjeta(tarjeta,idproyecto);

        } catch (Exception e) {
            FacesUtil.errorMessage(FacesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
        }
        return result;
    }

    // </editor-fold>
    // <editor-fold defaultstate="collapsed" desc="Long countLikeByDescripcion( String descripcion, Long idproyecto)">
    @Override
    public Long countLikeByDescripcion(String descripcion, Long idproyecto) {
        Long result = 0L;
        try {
            result = tarjetaRestClient.countLikeByDescripcion(descripcion, idproyecto);

        } catch (Exception e) {
            FacesUtil.errorMessage(FacesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
        }
        return result;
    }
    // </editor-fold>

    // <editor-fold defaultstate="collapsed" desc="public List<Tarjeta> likeByTarjeta(String tarjeta, Long idproyecto)">
    @Override
    public List<Tarjeta> likeByTarjeta(String tarjeta, Long idproyecto) {
        return tarjetaRestClient.likeByTarjeta(tarjeta,idproyecto);
    }
// </editor-fold>

    // <editor-fold defaultstate="collapsed" desc="public List<Tarjeta> likeByTarjetaPagination(String tarjeta, Pagination pagination, Long idproyecto)">
    @Override
    public List<Tarjeta> likeByTarjetaPagination(String tarjeta, Pagination pagination, Long idproyecto) {

        return tarjetaRestClient.likeByTarjetaPagination(tarjeta, pagination.getPage(), pagination.getSize(),idproyecto);

    }
// </editor-fold>

    // <editor-fold defaultstate="collapsed" desc="List<Tarjeta> likeByDescripcion(String descripcion, Long idproyecto)">
    @Override
    public List<Tarjeta> likeByDescripcion(String descripcion, Long idproyecto) {
        return tarjetaRestClient.likeByDescripcion(descripcion,idproyecto);
    }
// </editor-fold>

    // <editor-fold defaultstate="collapsed" desc="List<Tarjeta> betweenDate(@QueryParam("fechainicial") Date fechainicial, @QueryParam("fechafinal") Date fechafinal, Long idproyecto) ">
    public List<Tarjeta> betweenDate(@QueryParam("fechainicial") Date fechainicial, @QueryParam("fechafinal") Date fechafinal, Long idproyecto) {
        return tarjetaRestClient.betweenDate(fechainicial, fechafinal,idproyecto);
    }
    // </editor-fold>

    // <editor-fold defaultstate="collapsed" desc="List<Tarjeta> searchLikeByTarjeta(String tarjeta, Bson filter, Document sort, Integer page, Integer size, Long idproyecto)">
    @Override
    public List<Tarjeta> searchLikeByTarjeta(String tarjeta, Bson filter, Document sort, Integer page, Integer size, Long idproyecto) {
        List<Tarjeta> tarjetaList = new ArrayList<>();
        try {
            tarjetaList = tarjetaRestClient.searchLikeByTarjeta(tarjeta,
                    EncodeUtil.encodeBson(filter),
                    EncodeUtil.encodeBson(sort),
                    page, size,idproyecto);
        } catch (Exception e) {
            FacesUtil.errorMessage(FacesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
        }
        return tarjetaList;
    }
// </editor-fold>

    // <editor-fold defaultstate="collapsed" desc="Long searchCountLikeByTarjeta(String tarjeta, Bson filter, Document sort, Integer page, Integer size,, Long idproyecto)">
    @Override
    public Long searchCountLikeByTarjeta(String tarjeta, Bson filter, Document sort, Integer page, Integer size, Long idproyecto) {
        Long result = 0L;
        try {
            result = tarjetaRestClient.searchCountLikeByTarjeta(tarjeta,
                    EncodeUtil.encodeBson(filter),
                    EncodeUtil.encodeBson(sort),
                    page, size,idproyecto);
        } catch (Exception e) {
            FacesUtil.errorMessage(FacesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
        }
        return result;
    }
// </editor-fold>

// <editor-fold defaultstate="collapsed" desc="List<Tarjeta> searchLikeByDescripcion(String descripcion, Bson filter, Document sort, Integer page, Integer size, Long idproyecto)">
    @Override
    public List<Tarjeta> searchLikeByDescripcion(String descripcion, Bson filter, Document sort, Integer page, Integer size, Long idproyecto) {
        List<Tarjeta> tarjetaList = new ArrayList<>();
        try {
            tarjetaList = tarjetaRestClient.searchLikeByDescripcion(descripcion,
                    EncodeUtil.encodeBson(filter),
                    EncodeUtil.encodeBson(sort),
                    page, size,idproyecto);
        } catch (Exception e) {
            FacesUtil.errorMessage(FacesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
        }
        return tarjetaList;
    }
// </editor-fold>

    // <editor-fold defaultstate="collapsed" desc="Long searchCountLikeByDescripcion(String descripcion, Bson filter, Document sort, Integer page, Integer size, Long idproyecto)">
    @Override
    public Long searchCountLikeByDescripcion(String descripcion, Bson filter, Document sort, Integer page, Integer size, Long idproyecto) {
        Long result = 0L;
        try {
            result = tarjetaRestClient.searchCountLikeByDescripcion(descripcion,
                    EncodeUtil.encodeBson(filter),
                    EncodeUtil.encodeBson(sort),
                    page, size,idproyecto);
        } catch (Exception e) {
            FacesUtil.errorMessage(FacesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
        }
        return result;
    }
// </editor-fold>
    // <editor-fold defaultstate="collapsed" desc="List<Tarjeta> validarFechaFinalEsteSprintActual(List<Tarjeta> list, Sprint sprint)">

    /**
     * Valida y garantiza que la ultima fecha de la tarjeta este en el sprint
     * actual
     *
     * @param list
     * @param sprint
     * @return
     */
    @Override
    public List<Tarjeta> validarFechaFinalEsteSprintActual(List<Tarjeta> list, Sprint sprint) {
        List<Tarjeta> result = new ArrayList<>();
        try {
            if (list == null || list.isEmpty()) {
                return result;
            } else {
                /**
                 * Verifica que la fecha final de la tarjeta no sea menor que la
                 * fecha inicial del sprint si ocurre actualiza con la fecha y
                 * hora actual
                 */
                Integer count = 0;
                for (Tarjeta t : list) {

                    if (DateUtil.fechaMenor(t.getFechafinal(), sprint.getFechainicial())) {
                        t.setFechafinal(sprint.getFechafinal());
                        update(t);
                        list.get(0).setFechafinal(t.getFechafinal());
                    }
                    count++;
                }
            }
        } catch (Exception e) {
            FacesUtil.errorMessage(FacesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
        }
        return list;
    }

    // </editor-fold>
    // <editor-fold defaultstate="collapsed" desc="List<Tarjeta> validarFechaFinalSprintPlanTrabajo(List<Tarjeta> list, Sprint sprint)">
    /**
     * Valida y garantiza que la ultima fecha de la tarjeta en el sprint del
     * plan de trabajo actual
     *
     * @param list
     * @param sprint
     * @return
     */
    @Override
    public List<Tarjeta> validarFechaFinalSprintPlanTrabajo(List<Tarjeta> list, Sprint sprint) {
        List<Tarjeta> result = new ArrayList<>();
        try {
            if (list == null || list.isEmpty()) {
                return result;
            } else {
                /**
                 * Verifica que la fecha final de la tarjeta no sea menor que la
                 * fecha inicial del sprint si ocurre actualiza con la fecha y
                 * hora actual
                 */
                Integer count = 0;
                for (Tarjeta t : list) {
                    if (t.getIdproyecto().equals(sprint.getProyectoView().getIdproyecto())) {
                        if (DateUtil.fechaMenor(t.getFechafinal(), sprint.getFechainicial())) {
                            t.setFechafinal(sprint.getFechafinal());
                            t.setLastModification(t.getActionHistory().getLast().getFecha());
                            update(t);
                            list.get(count).setFechafinal(t.getFechafinal());
                        }else{
                          //  t.setLastModification(t.getActionHistory().getLast().getFecha());
                         //   list.get(count).setLastModification(t.getActionHistory().getLast().getFecha());
                        }

                    }
                    count++;
                }
            }
        } catch (Exception e) {
            FacesUtil.errorMessage(FacesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
        }
        return list;
    }

    // </editor-fold>
    // <editor-fold defaultstate="collapsed" desc="List<Tarjeta> orderListForIdTarjetaReserve(List<Tarjeta> list)">
    /**
     *
     * @param list
     * @return lista de tarjeta ordenada por idtarjeta en orden inverso
     */
    @Override
    public List<Tarjeta> orderListForIdTarjetaReserve(List<Tarjeta> list) {
        try {
            list = list.stream().sorted(Comparator.comparing(Tarjeta::getIdtarjeta).reversed()).collect(Collectors.toList());

        } catch (Exception e) {
            FacesUtil.errorMessage(FacesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
        }
        return list;
    }
    // </editor-fold>

    // <editor-fold defaultstate="collapsed" desc="Integer positionOfTarjeta(Tarjeta tarjeta, List<Tarjeta> tarjetas)">
    @Override
    public Integer positionOfTarjeta(Tarjeta tarjeta, List<Tarjeta> tarjetas) {
        Integer result = -1;
        try {

            Integer c = 0;
            for (Tarjeta t : tarjetas) {
                if (t.getIdtarjeta().equals(tarjeta.getIdtarjeta())) {
                    result = c;
                    break;
                }
                c++;

            }
        } catch (Exception e) {
            FacesUtil.errorMessage(FacesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
        }
        return result;
    }
    // </editor-fold>

    // <editor-fold defaultstate="collapsed" desc=" public TotalesTarjetasEstadistica calcularTotalesTarjetasEstadistica(Long iduser, Long idproyecto)">

    @Override
    public TotalesTarjetasEstadistica calcularTotalesTarjetasEstadistica(Long iduser, Long idproyecto) {
        TotalesTarjetasEstadistica result = new TotalesTarjetasEstadistica(0L, 0L, 0L,0L, iduser);
        try {
            Integer page = 0;
            Integer size = 0;
            Document sortTarjeta = new Document("idtarjeta", 1);
            Document sortSprint = new Document("idtarjeta", 1); 
            /**
             * CargarTarjetas
             */
            Bson filter0 = eq("user.iduser", iduser);
            Bson filter = and(filter0, eq("active", Boolean.TRUE),
                    eq("columna", "pendiente"),
                    ne("idsprint",0L));

            Long pendiente = tarjetaServices.count(filter, sortTarjeta, page, size,idproyecto);

            Bson filterProgreso = and(filter0, eq("active", Boolean.TRUE),
                    eq("columna", "progreso"));

            Long progreso = tarjetaServices.count(filterProgreso, sortTarjeta, page, size,idproyecto);

            Bson filterFinalizado = and(filter0, eq("active", Boolean.TRUE),
                    eq("columna", "finalizado"));

            Long finalizado = tarjetaServices.count(filterFinalizado, sortTarjeta, page, size,idproyecto);

            Bson filterBacklog = and(filter0, eq("active", Boolean.TRUE),
                    eq("backlog", true),
                            eq("idsprint",0L));

            Long backlog = tarjetaServices.count(filterBacklog, sortTarjeta, page, size,idproyecto);
            
          result.setTotalTarjetasBacklog(backlog);
          result.setTotalTarjetasFinalizado(finalizado);
          result.setTotalTarjetasPendiente(pendiente);
          result.setTotalTarjetasProgreso(progreso);

            
     } catch (Exception e) {
            FacesUtil.errorMessage(FacesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
        }
        return result;
    }
    
    // </editor-fold>



    
     
}


```



### En una clase Faces

```java
@Named
@ViewScoped
@Data
public class TarjetasFaces implements Serializable, JmoordbCoreXHTMLUtil, IPaginator, SprintFacesServices, FacesServices {

    @Inject
    TarjetaServices tarjetaServices;

 public void save(Tarjeta tarjeta) {
   Optional<Tarjeta> tarjetaSaved = tarjetaServices.save(tarjeta);
            if (!tarjetaSaved.isPresent()) {

                FacesUtil.warningDialog(rf.fromCore("warning.warning"), rf.fromCore("warning.save"));

            } else {
                tarjeta.setIdtarjeta(tarjetaSaved.get().getIdtarjeta());
}
}

  public String nextProgreso(Tarjeta tarjeta) {
 Tarjeta tarjetaDB = tarjetaServices.findByIdtarjetaIdproyecto(tarjeta.getIdtarjeta(), proyectoSelected.getIdproyecto()).get();
}

```



## Bases de datos y colecciones dinamicas en base a campos fechas

En algunas ocasiones deseamos generar bases de datos y colecciones dinámicas en base a una fecha.

Por ejemplo generar una base datos para cada año y las colecciones por mes y por cada empresa. De manera que contamos con una mejor clasificación de los documentos , lo que genera un mejor desempeño de la aplicación al distribuir los documentos en varias bases de datos y colecciones.

En el ejemplo hipotético asuma que cuenta con un modelo como el siguiente:

```java
@Entity
public class Venta {

    @Id(strategy = GenerationType.AUTO)
    private Long idventa;

    @Column
    private Long idempresa;

    @Column
    private Date fechaHora;
    @Column
    private Double total;
....
}

```


```java
@Entity
public class Empresa {
    @Id 
    private Long idempresa;
    @Column
    private String empresa;

....
  }


```


### Repositorio


```java

@Repository(database = "{mongodb.database1}", entity = Empresa.class)
public interface EmpresaRepository extends CrudRepository<Empresa, Long> {

    @Lookup
    public List<Empresa> lookup(Search search);

    @Find
    public List<Empresa> findByEmpresa(String empresa);
    
    @Count()
    public Long count(Search... search);
    
    @CountLikeBy(caseSensitive = CaseSensitive.NO, likeByType = LikeByType.ANYWHERE)
    public Long countLikeByEmpresa(String empresa);
       
    @CountLikeBy(caseSensitive = CaseSensitive.NO, likeByType = LikeByType.ANYWHERE)
    public Long countLikeByDescripcion(String descripcion);

    @SearchCountLikeBy(caseSensitive = CaseSensitive.NO, likeByType = LikeByType.ANYWHERE)
    public Long searchCountLikeByEmpresa(String empresa, Search search);
    
    @SearchCountLikeBy(caseSensitive = CaseSensitive.NO, likeByType = LikeByType.ANYWHERE)
    public Long searchCountLikeByDescripcion(String descripcion, Search search);
    
    @LikeBy(caseSensitive = CaseSensitive.NO, typeOrder = TypeOrder.ASC, likeByType = LikeByType.ANYWHERE)
    public List<Empresa> likeByEmpresa(String empresa);
    
    @LikeBy(caseSensitive = CaseSensitive.NO, typeOrder = TypeOrder.ASC, likeByType = LikeByType.ANYWHERE)
    public List<Empresa> likeByEmpresaPagination(String empresa, Pagination pagination);
    
   
    
    @LikeBy(caseSensitive = CaseSensitive.NO, typeOrder = TypeOrder.ASC, likeByType = LikeByType.ANYWHERE)
    public List<Empresa> likeByDescripcion(String descripcion);


   @SearchLikeBy(caseSensitive = CaseSensitive.NO, typeOrder = TypeOrder.ASC, likeByType = LikeByType.ANYWHERE)
    public List<Empresa> searchLikeByEmpresa(String empresa, Search search);

    @SearchLikeBy(caseSensitive = CaseSensitive.NO, typeOrder = TypeOrder.ASC, likeByType = LikeByType.ANYWHERE)
    public List<Empresa> searchLikeByDescripcion(String descripcion, Search search);

   
}


```

**VentaRepository**

```java

@Repository(database = "{mongodb.database1}", entity = Venta.class)
public interface VentaRepository extends CrudRepository<Venta, Long> {

    @Lookup
    public List<Venta> lookup(Search search);

    @Count()
    public Long count(Search... search);
    
    @CountLikeBy(caseSensitive = CaseSensitive.NO, likeByType = LikeByType.ANYWHERE)
    public Long countLikeByVenta(String venta);
       
    @CountLikeBy(caseSensitive = CaseSensitive.NO, likeByType = LikeByType.ANYWHERE)
    public Long countLikeByDescripcion(String descripcion);

    @SearchCountLikeBy(caseSensitive = CaseSensitive.NO, likeByType = LikeByType.ANYWHERE)
    public Long searchCountLikeByVenta(String venta, Search search);
    
    @SearchCountLikeBy(caseSensitive = CaseSensitive.NO, likeByType = LikeByType.ANYWHERE)
    public Long searchCountLikeByDescripcion(String descripcion, Search search);
    
    @LikeBy(caseSensitive = CaseSensitive.NO, typeOrder = TypeOrder.ASC, likeByType = LikeByType.ANYWHERE)
    public List<Venta> likeByVenta(String venta);
    
    @LikeBy(caseSensitive = CaseSensitive.NO, typeOrder = TypeOrder.ASC, likeByType = LikeByType.ANYWHERE)
    public List<Venta> likeByVentaPagination(String venta, Pagination pagination);
    
   
    
    @LikeBy(caseSensitive = CaseSensitive.NO, typeOrder = TypeOrder.ASC, likeByType = LikeByType.ANYWHERE)
    public List<Venta> likeByDescripcion(String descripcion);


   @SearchLikeBy(caseSensitive = CaseSensitive.NO, typeOrder = TypeOrder.ASC, likeByType = LikeByType.ANYWHERE)
    public List<Venta> searchLikeByVenta(String venta, Search search);

    @SearchLikeBy(caseSensitive = CaseSensitive.NO, typeOrder = TypeOrder.ASC, likeByType = LikeByType.ANYWHERE)
    public List<Venta> searchLikeByDescripcion(String descripcion, Search search);

    @Find
    public List<Venta> findByFechahoraGreaterThanEqualAndFechahoraLessThanEqual(@ExcludeTime Date start, @ExcludeTime Date end);


}

```

## Crear el controller para empresa

```java
@Path("empresa")

public class EmpresaController {

    // <editor-fold defaultstate="collapsed" desc="Inject">
    @Inject
    EmpresaRepository empresaRepository;

    

// </editor-fold>
    // <editor-fold defaultstate="collapsed" desc="findAll">
    @GET
    @Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})
       public List<Empresa> findAll() {
        List<Empresa> empresaList = new ArrayList<>();

        return empresaRepository.findAll();
    }
// </editor-fold>

    // <editor-fold defaultstate="collapsed" desc="Empresa findByIdempresa">
    @GET
    @Path("{idempresa}")
    public Empresa findByIdempresa(
            @Parameter(description = "El idempresa", required = true, example = "1", schema = @Schema(type = SchemaType.NUMBER)) @PathParam("idempresa") Long idempresa) {

        return empresaRepository.findByPk(idempresa).orElseThrow(
                () -> new WebApplicationException("No hay empresa con idempresa " + idempresa, Response.Status.NOT_FOUND));

    }
// </editor-fold>

    // <editor-fold defaultstate="collapsed" desc="Empresa findByEmpresa">
    @GET
    @Path("empresa")
    public List<Empresa> findByEmpresa(@Parameter(description = "El empresa", required = true, example = "1", schema = @Schema(type = SchemaType.STRING)) @QueryParam("empresa") final String empresa) {

        return empresaRepository.findByEmpresa(empresa);

    }
//// </editor-fold>
    // <editor-fold defaultstate="collapsed" desc="Response save">

    @POST
    public Response save(
            @RequestBody(description = "Crea un nuevo empresa.", content = @Content(mediaType = "application/json", schema = @Schema(implementation = Empresa.class))) Empresa empresa) {

        Optional<Empresa> empresaOptional = empresaRepository.save(empresa);
        if (empresaOptional.isPresent()) {
//            saveHistory(empresa);
            return Response.status(201).entity(empresaOptional.get()).build();
        } else {
            return Response.status(400).entity("Error " + empresaRepository.getJmoordbException().getLocalizedMessage()).build();
        }
    }
// </editor-fold>
    // <editor-fold defaultstate="collapsed" desc="Response update">

    @PUT
    public Response update(
            @RequestBody(description = "Crea un nuevo empresa.", content = @Content(mediaType = "application/json", schema = @Schema(implementation = Empresa.class))) Empresa empresa) {

        if (empresaRepository.update(empresa)) {
            return Response.status(201).entity(empresa).build();
        } else {

            return Response.status(400).entity("Error " + empresaRepository.getJmoordbException().getLocalizedMessage()).build();

        }
    }
// </editor-fold>

    // <editor-fold defaultstate="collapsed" desc="Response delete">
    @DELETE
    @Path("{idempresa}")
    public Response delete(
            @Parameter(description = "El elemento idempresa", required = true, example = "1", schema = @Schema(type = SchemaType.NUMBER)) @PathParam("idempresa") Long idempresa) {

        if (empresaRepository.deleteByPk(idempresa) == 0L) {
            return Response.status(201).entity(Boolean.TRUE).build();
        } else {
            return Response.status(400).entity("Error " + empresaRepository.getJmoordbException().getLocalizedMessage()).build();
        }
    }
    // </editor-fold>


    // <editor-fold defaultstate="collapsed" desc="List<User> lookup(@QueryParam("filter") String filter, @QueryParam("sort") String sort, @QueryParam("page") Integer page, @QueryParam("size") Integer size)">
    @GET
    @Path("lookup")
    @Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})
    public List<Empresa> lookup(@QueryParam("filter") String filter, @QueryParam("sort") String sort, @QueryParam("page") Integer page, @QueryParam("size") Integer size) {
        List<Empresa> suggestions = new ArrayList<>();
        try {

            Search search = DocumentUtil.convertForLookup(filter, sort, page, size);

            suggestions = empresaRepository.lookup(search);

        } catch (Exception e) {

            MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + "error: " + e.getLocalizedMessage());
        }

        return suggestions;
    }

    // </editor-fold>
    
    // <editor-fold defaultstate="collapsed" desc="Long count(@QueryParam("filter") String filter, @QueryParam("sort") String sort, @QueryParam("page") Integer page, @QueryParam("size") Integer size)">
    @GET
    @Path("count")
    @Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})
    public Long count(@QueryParam("filter") String filter, @QueryParam("sort") String sort, @QueryParam("page") Integer page, @QueryParam("size") Integer size) {
        Long result = 0L;
        try {

            Search search = DocumentUtil.convertForLookup(filter, sort, page, size);
            result = empresaRepository.count(search);

        } catch (Exception e) {

            MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + "error: " + e.getLocalizedMessage());
        }

        return result;
    }

    // </editor-fold>
  
    // <editor-fold defaultstate="collapsed" desc="Long countLikeEmpresa(@QueryParam("empresa") String empresa)">
    @GET
    @Path("countlikebyempresa")
    @Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})
    public Long countLikeEmpresa(@QueryParam("empresa") String empresa) {
        Long result = 0L;
        try {
            result = empresaRepository.countLikeByEmpresa(empresa);
        } catch (Exception e) {
            MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + "error: " + e.getLocalizedMessage());
        }

        return result;
    }

    // </editor-fold>
    // <editor-fold defaultstate="collapsed" desc=" List<Empresa> likeByName(@QueryParam("empresa") String empresa)">
    @GET
    @Path("likebyempresa")
    @Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})
    public List<Empresa> likeByName(@QueryParam("empresa") String empresa) {
        List<Empresa> suggestions = new ArrayList<>();
        try {
            suggestions = empresaRepository.likeByEmpresa(empresa);
        } catch (Exception e) {
            MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + "error: " + e.getLocalizedMessage());
        }
        return suggestions;
    }

    // </editor-fold>
}


```




Insertar registros en la coleccion Empresa

```shell
curl --location --request POST 'http://localhost:9000/capitulo19/api/empresa/' --header 'Content-Type: application/json' --data-raw '{"idempresa": 1, "pais": "Empresa X"}'

curl --location --request POST 'http://localhost:9000/capitulo19/api/empresa/' --header 'Content-Type: application/json' --data-raw '{"idempresa": 2, "pais": "Advanced Tec."}'

```

Consultas


```shell

curl --location --request GET http://localhost:9000/capitulo19/api/venta

```



### Manejo de Ventas

Puede observar que los metodos save y update() revisen como parametro la entidad y de esta debemos tomar la fecha convertida mediante

```java
 Date dateconverter = JmoordbCoreDateUtil.dateToiSODateToDate(venta.getFechaHora());

```

Y para los metodos de consulta se pasa como parametros el año y mes.




VentaController

```java
@Path("venta")
@RequestScoped
public class VentaController implements Serializable {

    private String nameOfCollection = "venta_empresa_";
    // <editor-fold defaultstate="collapsed" desc="Inject">
    @Inject
    VentaRepository ventaRepository;
    @Inject
    EmpresaRepository empresaRepository;

// </editor-fold>
    // <editor-fold defaultstate="collapsed" desc="Venta findByIdventaIdproyecto(@QueryParam("idventa") Long idventa,       @QueryParam("anio") Long anio  ) ">
    @GET
    @RolesAllowed({"admin"})
    @Path("idventaanio")
    
    public Venta findByIdventa(@QueryParam("idventa") Long idventa, @QueryParam("anio") Long anio) {

        ventaRepository.setDynamicCollection(nameOfCollection + anio);
        return ventaRepository.findByPk(idventa).orElseThrow(
                () -> new WebApplicationException("No hay venta con idventa " + idventa, Response.Status.NOT_FOUND));

    }
// </editor-fold>

    // <editor-fold defaultstate="collapsed" desc="Response save">
    @POST
    public Response save(
            @RequestBody(description = "Crea un nuevo venta.", content = @Content(mediaType = "application/json", schema = @Schema(implementation = Venta.class))) Venta venta) {

        Boolean conIsoDate = Boolean.TRUE;
        if (conIsoDate) {
            
            Date dateconverter = JmoordbCoreDateUtil.dateToiSODateToDate(venta.getFechaHora());
            
            ventaRepository.setDynamicDatabase("transaccion_" + JmoordbCoreDateUtil.anioDeUnaFecha(dateconverter).toString().trim() + "db");
            Integer numeroMes = JmoordbCoreDateUtil.mesDeUnaFechaStartEneroWith0(dateconverter);
        
            ventaRepository.setDynamicCollection(nameOfCollection + venta.getIdempresa().toString().trim() + "_" + JmoordbCoreDateUtil.getNombreMes(numeroMes).toLowerCase());

        } else {

            ventaRepository.setDynamicDatabase("transaccion_" + JmoordbCoreDateUtil.anioDeUnaFecha(venta.getFechaHora()).toString().trim() + "db");
            Integer numeroMes = JmoordbCoreDateUtil.mesDeUnaFechaStartEneroWith0(venta.getFechaHora());
            ventaRepository.setDynamicCollection(nameOfCollection + venta.getIdempresa().toString().trim() + "_" + JmoordbCoreDateUtil.getNombreMes(numeroMes).toLowerCase());

        }

        Optional<Venta> ventaOptional = ventaRepository.save(venta);
        if (ventaOptional.isPresent()) {

            return Response.status(201).entity(ventaOptional.get()).build();
        } else {
            return Response.status(400).entity("Error " + ventaRepository.getJmoordbException().getLocalizedMessage()).build();
        }

    }
// </editor-fold>

    // <editor-fold defaultstate="collapsed" desc="Response update">
    @PUT
    public Response update(
            @RequestBody(description = "Crea un nuevo venta.", content = @Content(mediaType = "application/json", schema = @Schema(implementation = Venta.class))) Venta venta) {
        Boolean conIsoDate = Boolean.TRUE;
        if (conIsoDate) {
            Date dateconverter = JmoordbCoreDateUtil.dateToiSODateToDate(venta.getFechaHora());

            ventaRepository.setDynamicDatabase("transaccion_" + JmoordbCoreDateUtil.anioDeUnaFecha(dateconverter).toString().trim() + "db");
            Integer numeroMes = JmoordbCoreDateUtil.mesDeUnaFechaStartEneroWith0(dateconverter);
            ventaRepository.setDynamicCollection(nameOfCollection + venta.getIdempresa().toString().trim() + "_" + JmoordbCoreDateUtil.getNombreMes(numeroMes).toLowerCase());

        } else {

            ventaRepository.setDynamicDatabase("transaccion_" + JmoordbCoreDateUtil.anioDeUnaFecha(venta.getFechaHora()).toString().trim() + "db");
            Integer numeroMes = JmoordbCoreDateUtil.mesDeUnaFechaStartEneroWith0(venta.getFechaHora());
            ventaRepository.setDynamicCollection(nameOfCollection + venta.getIdempresa().toString().trim() + "_" + JmoordbCoreDateUtil.getNombreMes(numeroMes).toLowerCase());

        }

        if (ventaRepository.update(venta)) {

            return Response.status(201).entity(venta).build();
        } else {
            System.out.println("\t>>>>>>>> [error]" + MessagesUtil.nameOfClassAndMethod());
            System.out.println("\t>>>>>>>> [error]" + MessagesUtil.nameOfClassAndMethod() + " [error] " + ventaRepository.getJmoordbException().getLocalizedMessage());
            return Response.status(400).entity("Error " + ventaRepository.getJmoordbException().getLocalizedMessage()).build();
        }
    }
// </editor-fold>

    // <editor-fold defaultstate="collapsed" desc="Response delete(@QueryParam("idventa") Long idventa, @QueryParam("anio") Integer anio, @QueryParam("mes") Integer mes) ">
    @DELETE
    @Path("idventaanio")
    public Response delete(@QueryParam("idventa") Long idventa, @QueryParam("anio") Integer anio, @QueryParam("mes") Integer mes) {

        ventaRepository.setDynamicDatabase("transaccion_" + anio.toString().trim() + "db");
        Integer numeroMes = mes;
        ventaRepository.setDynamicCollection(nameOfCollection + idventa.toString().trim() + "_" + JmoordbCoreDateUtil.getNombreMes(numeroMes).toLowerCase());

        if (ventaRepository.deleteByPk(idventa) == 0L) {
            return Response.status(201).entity(Boolean.TRUE).build();
        } else {
            return Response.status(400).entity("Error " + ventaRepository.getJmoordbException().getLocalizedMessage()).build();
        }
    }

    // </editor-fold>
    // <editor-fold defaultstate="collapsed" desc="Response deleteMany(@QueryParam("filter") String filter @QueryParam("anio") Integer anio, @QueryParam("mes") Integer mes)">
    @DELETE
    @Path("deletemany")
    public Response deleteMany(@QueryParam("filter") String filter, @QueryParam("idempresa") Long idempresa, @QueryParam("anio") Integer anio, @QueryParam("mes") Integer mes) {
        ventaRepository.setDynamicDatabase("transaccion_" + anio.toString().trim() + "db");
        Integer numeroMes = mes;
        ventaRepository.setDynamicCollection(nameOfCollection + idempresa.toString().trim() + "_" + JmoordbCoreDateUtil.getNombreMes(numeroMes).toLowerCase());

        Search search = DocumentUtil.convertForLookup(filter, "", 0, 0);
        if (ventaRepository.deleteMany(search) == 0L) {
            return Response.status(201).entity(Boolean.TRUE).build();
        } else {
            return Response.status(400).entity("Error " + ventaRepository.getJmoordbException().getLocalizedMessage()).build();
        }
    }
    // </editor-fold>

    // <editor-fold defaultstate="collapsed" desc="List<Venta> lookup(@QueryParam("filter") String filter, @QueryParam("sort") String sort, @QueryParam("page") Integer page, @QueryParam("size") Integer size,@QueryParam("idempresa") Long idempresa, @QueryParam("anio") Integer anio, @QueryParam("mes") Integer mes)">
    @GET
    @Path("lookup")
    @Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})
    public List<Venta> lookup(@QueryParam("filter") String filter, @QueryParam("sort") String sort, @QueryParam("page") Integer page, @QueryParam("size") Integer size, @QueryParam("idempresa") Long idempresa, @QueryParam("anio") Integer anio, @QueryParam("mes") Integer mes) {
        List<Venta> suggestions = new ArrayList<>();
        try {

            ventaRepository.setDynamicDatabase("transaccion_" + anio.toString().trim() + "db");
            Integer numeroMes = mes;
            ventaRepository.setDynamicCollection(nameOfCollection + idempresa.toString().trim() + "_" + JmoordbCoreDateUtil.getNombreMes(numeroMes).toLowerCase());

            Search search = DocumentUtil.convertForLookup(filter, sort, page, size);

            suggestions = ventaRepository.lookup(search);

        } catch (Exception e) {

            MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + "error: " + e.getLocalizedMessage());
        }
        System.out.println("Resultado: " + suggestions);
        return suggestions;
    }

    // </editor-fold>
    // <editor-fold defaultstate="collapsed" desc="Long count(@QueryParam("filter") String filter, @QueryParam("sort") String sort, @QueryParam("page") Integer page, @QueryParam("size") Integer size, @QueryParam("anio") Long anio)">
    @GET
    @Path("count")
    @Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})
    public Long count(@QueryParam("filter") String filter, @QueryParam("sort") String sort, @QueryParam("page") Integer page, @QueryParam("size") Integer size, @QueryParam("idempresa") Long idempresa, @QueryParam("anio") Integer anio, @QueryParam("mes") Integer mes) {
        Long result = 0L;
        try {
            ventaRepository.setDynamicDatabase("transaccion_" + anio.toString().trim() + "db");
            Integer numeroMes = mes;
            ventaRepository.setDynamicCollection(nameOfCollection + idempresa.toString().trim() + "_" + JmoordbCoreDateUtil.getNombreMes(numeroMes).toLowerCase());

            Search search = DocumentUtil.convertForLookup(filter, sort, page, size);
            result = ventaRepository.count(search);

        } catch (Exception e) {

            MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + "error: " + e.getLocalizedMessage());
        }

        return result;
    }

    // </editor-fold>
    // <editor-fold defaultstate="collapsed" desc="List<Venta> findLastVenta @QueryParam("filter") String filter, @QueryParam("sort") String sort, @QueryParam("anio") Integer anio, @QueryParam("mes") Integer mes">
    @GET
    @Path("findlastventa")
    @Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})
    public List<Venta> findLastVenta(@QueryParam("filter") String filter, @QueryParam("sort") String sort, @QueryParam("idempresa") Long idempresa, @QueryParam("anio") Integer anio, @QueryParam("mes") Integer mes) {
        List<Venta> suggestions = new ArrayList<>();
        try {
            Integer numeroMes = mes;

            ventaRepository.setDynamicDatabase("transaccion_" + anio.toString().trim() + "db");
            ventaRepository.setDynamicCollection(nameOfCollection + idempresa.toString().trim() + "_" + JmoordbCoreDateUtil.getNombreMes(numeroMes).toLowerCase());
            Search search = DocumentUtil.convertForLookup(filter, sort, 1, 1);

            suggestions = ventaRepository.lookup(search);

        } catch (Exception e) {

            MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + "error: " + e.getLocalizedMessage());
        }

        return suggestions;
    }

    // </editor-fold>
    // <editor-fold defaultstate="collapsed" desc="List<Venta> findAllLastVenta @QueryParam("filter") String filter, @QueryParam("sort") String sort, @QueryParam("anio") Integer anio, @QueryParam("mes") Integer mes">
    @GET
    @Path("findalllastventa")
    @Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})

    public List<Venta> findAllLastVenta(@QueryParam("filter") String filter, @QueryParam("sort") String sort, @QueryParam("anio") Integer anio, @QueryParam("mes") Integer mes) {
        List<Venta> suggestions = new ArrayList<>();
        List<Venta> resultSearch = new ArrayList<>();
        try {
            Integer numeroMes = mes;
            Long countEstaciones = empresaRepository.count();

            for (int index = 1; index <= countEstaciones; index++) {

                ventaRepository.setDynamicDatabase("transaccion_" + anio.toString().trim() + "db");
                ventaRepository.setDynamicCollection(nameOfCollection + String.valueOf(index) + "_" + JmoordbCoreDateUtil.getNombreMes(numeroMes).toLowerCase());
                Search search = DocumentUtil.convertForLookup(filter, sort, 1, 1);

                resultSearch = ventaRepository.lookup(search);
                suggestions.addAll(resultSearch);
            }

        } catch (Exception e) {

            MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + "error: " + e.getLocalizedMessage());
        }

        return suggestions;
    }
    // </editor-fold>

    // <editor-fold defaultstate="collapsed" desc=" List<Venta> betweenDate(@QueryParam("fechainicial") Date fechainicial, @QueryParam("fechafinal") Date fechafinal, @QueryParam("anio") Long anio)">
    @GET
    @Path("betweendate")
    @Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})
    public List<Venta> betweenDate(@QueryParam("fechainicial") @DateFormat("dd-MM-yyyy") final Date fechainicial, @QueryParam("fechafinal") @DateFormat("dd-MM-yyyy") final Date fechafinal, @QueryParam("idempresa") Long idempresa, @QueryParam("anio") Integer anio, @QueryParam("mes") Integer mes) {

        List<Venta> suggestions = new ArrayList<>();
        try {
            ventaRepository.setDynamicDatabase("transaccion_" + anio.toString().trim() + "db");
            Integer numeroMes = mes;
            ventaRepository.setDynamicCollection(nameOfCollection + idempresa.toString().trim() + "_" + JmoordbCoreDateUtil.getNombreMes(numeroMes).toLowerCase());

            suggestions = ventaRepository.findByFechahoraGreaterThanEqualAndFechahoraLessThanEqual(fechainicial, fechafinal);

        } catch (Exception e) {

            MessagesUtil.error(MessagesUtil.nameOfClassAndMethod() + "error: " + e.getLocalizedMessage());
        }
        System.out.println("Resultado: " + suggestions);

        return suggestions;
    }

    // </editor-fold>
}


``


## Insertar registros en la coleccion ventas

```shell
curl --location --request POST 'http://localhost:9000/capitulo19/api/venta/' --header 'Content-Type: application/json' --data-raw '{"idventa":1,"idempresa": 1, "fechaHora": "2024-02-01T01:00:00Z", "total": 150.20}'



```

Crea la base de datos **transaccion_2024db" y crea la colección **venta_empresa_1_febrero**


Ejecute
```shell
curl --location --request POST 'http://localhost:9000/capitulo19/api/venta/' --header 'Content-Type: application/json' --data-raw '{"idventa":2,"idempresa": 2, "fechaHora": "2025-10-08T12:00:00Z", "total": 258.40}'



```
Crea la base de datos **transaccion_2025db" y crea la colección **venta_empresa_2_octubre**

Como puede observar genera la base de datos en base al año de la fecha pasada en la entidad Ventas, y los nombres de colección se genera en base al numero de mes obtenido de la fecha.

Esto ayuda a implementar el concepto de Sharding de MongoDB.


## Resumen


Este capítulo aborda el uso de la generación dinamica de bases de datos y colecciones.








