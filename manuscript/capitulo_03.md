# Capítulo 3

En este capítulo proporciona una guía para definir entidades utilizando Jmoordbcore.

Este capítulo incluye los temas:

* Entidades

* @Entity

* @Id

* @Column

* @DocumentEmbeddable

* @Embedded

* @Referenced

* Referenciados y Embebidos

* @Ignore

* @Domain

* @ViewEntity 


## Entidades

 La especificación [Jakarta Persistence](https://jakarta.ee/specifications/persistence/3.1/jakarta-persistence-spec-3.1.html#entities) define una entidad de la siguiente manera:


**"Una entidad es un objeto de dominio persistente y ligero."**



## @Entity

En Jmoordbcore, las entidades se definen mediante la anotación @Entity. Una entidad puede incluir atributos, documentos embebidos, referencias a otros documentos o a vistas.

Joordbcore admite colecciones Java de tipo List<>, Set<> o Stream<>.

La anotación @Entity se define de la siguiente manera:
 
```java

@Documented
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface Entity {
    String collection() default "";
    String database() default "{mongodb,database}";
    JakartaSource jakartaSource() default JakartaSource.JAKARTA;
    String commentary() default "";
}


```
 
* Collection y database: Son opcionales, sus valores se obtendrán del archivo micropofile-config.properties.

* JakartaSource: Determina si se utilizará Jakarta EE o una versión anterior de Java EE en el proyecto.

* Commentary: Especifica un comentario.

**Una entidad en Jmoordb incorpora las siguientes anotaciones:**

```java

@Entity  
@Id  
@AutoImplement
@DocumentEmbeddable
@Ignore
@Column  
@Embedded  
@Referenced  
@ViewEntity

```


## @Id

La anotación @Id se utiliza para indicar la clave primaria. Solo admite tipos de datos: **String o Long**.

Además, permite especificar la estrategia de generación para valores incrementales. Por ejemplo, puedes utilizar:

* @Id(strategy =  GenerationType.AUTO)  : Activa el autoincrementable, solo para atributos de tipo Long.

* @Id(strategy =  GenerationType.NONE) : Predeterminado, no usa autoincrementable.

* @Id(strategy = GenerationType.OBJECTID) : Utiliza el atributo _id de MongoDB como campo llave.

* @Id(strategy = GenerationType.UUID) : Utiliza genera valores UUID para el campo llave.

Cuando se utilice strategy = GenerationType.AUTO), debe definir una interfaz de tipo @AutoincrementRepository, en su paquete de repositorios.


La interface Id está definida de la siguiente manera:

```java

@Documented
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Id {
    String value() default "";
    GenerationType strategy() default  GenerationType.NONE;
    String commentary() default "";
}

```
 
A continuación, exploraremos ejemplos de cómo se utiliza @Id:


* Ejemplo de cómo definir una entidad con una clave primaria de tipo String:


 
```java
@Entity
public class Oceano {
    @Id
    private String idoceano;
   
    public Oceano() {
    }
    //set/get
}
```
 


* Ejemplo de cómo definir una entidad con clave primaria de tipo Long, en la que se habilita la autogeneración automática de valores para el atributo idpais.


 
```java

@Entity
public class Pais {

 @Id(strategy = GenerationType.AUTO)
 private Long idpais;
 
}

```

 
## @Column

La anotación @Column mapea un atributo de la entidad a un atributo de un documento en una colección en MongoDB. Los atributos deben ser de tipos de datos válidos en Java.

La interface Column está definida de la siguiente manera:
 
```java

@Documented
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Column {
    String value() default "";
    String commentary() default "";
    boolean generateQuery() default false;
}

```

**Nota:** generateQuery se utiliza en conjunto con el plugin para NetBeans jmoordbcoreplugin para la generación automática de opciones de búsqueda por ese atributo.


A continuación, exploraremos un ejemplo definición de un atributo de tipo String y otro de tipo Date respectivamente.

```java

@Entity
public class Pais {
  @Id(strategy = GenerationType.AUTO)
  private Long idpais;
  
  @Column
  private String pais;
  
  @Column
  private Date fecha;
}

```



## @DocumentEmbeddable

La clase que se defina mediante la anotación @DocumentEmbeddable se incrustara dentro de otra entidad. No es obligatorio que tengan una llave primaria definida con la anotación @Id.

Esta clase solo existe dentro del contexto de la entidad que la contiene. Para indicar que será embebida, se utiliza la anotación @Embedded.

Las entidades de tipo DocumentEmbeddable, pueden contener documentos embebidos y documentos referenciados. 


La interface DocumentEmbeddable se define de la siguiente manera:


```java

@Retention(value = RetentionPolicy.RUNTIME)
@Target(value = {ElementType.TYPE})
public @interface DocumentEmbeddable {

    public String collection() default "";

    public String database() default "{mongodb,database}";

    public JakartaSource jakartaSource() default JakartaSource.JAKARTA;

    public String commentary() default "";
}


```

A continuación se muestran ejemplos del uso de @DocumentEmbeddable:


* Ejemplo de definición de una clase de tipo @DocumentEmbeddable simple:

 
```java


@DocumentEmbeddable
public class Lenguajes {       
       @Column
       private String lenguaje;
       @Column
       private Integer preferencia;
}

```

* Ejemplo de definición de una clase de tipo @DocumentEmbeddable que incluye referencias a otras colecciones:
 
```java


@DocumentEmbeddable
public class Idioma {
       @Id
       private String ididioma;
       @Column
       private String idioma;
       @Referenced(from = "cultura", localField = "idcultura")
       private Cultura cultura;
}

```
 


## @Embedded


La anotación @Embedded se utiliza en una entidad para indicar que la clase es un documento embebido declarado con la anotación @DocumentEmbeddable o @Entity.

La interface Embedded está definida de la siguiente manera:

```java
@Retention(RetentionPolicy.RUNTIME)
public @interface Embedded {
    String name() default "";
    String commentary() default "";
}


```
 

En el siguiente ejemplo se muestra una entidad con documentos embebidos:


```java

@Entity(jakartaSource = JakartaSource.JAKARTA)
public class Habitante {
    @Id
    private String idhabitante;

    @Embedded
    Idioma idioma;
  
    @Embedded
    Set<Persona> persona;
       
    @Embedded
    List<Deporte> deporte;
}
```


En el ejemplo, se puede observar cómo la entidad Persona se utiliza como una clase embebida dentro de la entidad Habitante:

```java

@Entity(jakartaSource = JakartaSource.JAKARTA)
public class Persona {
    @Id
    private String idpersona;
    @Column
    private String nombre;
}

```


## @Referenced

La anotación @Referenced se utiliza para definir colecciones referenciadas, que son similares a las relaciones en bases de datos relacionales. 

Esta anotación admite que los datos de retorno sean tipo entidad, o colecciones como List<>, Stream<>, Set<>. 

En el capítulo @Repository se profundiza en la sintaxis y uso de esta anotación. 



La interface Referenced está definida de la siguiente manera:


 
```java

@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD, ElementType.FIELD})
public @interface Referenced {
 String from();
 String localField();
 TypeReferenced typeReferenced() default TypeReferenced.REFERENCED;
 String commentary() default "";
}


```
 

La anotación @Referenced incluye los siguientes atributos:

* from: Es la colección referenciada.

* localField: Campo local que hace referencia a la colección referenciada. 

* TypeReferenced (Predeteminado **ReferenceType.REFERENCED**): Busca en las colecciones referenciadas mediante id.


El enum TypeReferenced está definido de la siguiente manera:
 
```java

public enum TypeReferenced { 
   REFERENCED, EMBEDDED     
}

```
 
En el siguiente ejemplo se muestra una entidad con documentos referenciados: 


```java

@Entity(jakartaSource = JakartaSource.JAKARTA)
public class Habitante {
    @Id
    private String idhabitante;

    @Referenced(from = "pais", localField = "idpais")
    Set<Pais> pais;

    @Referenced(from = "deporte", localField = "iddeporte")
    List<Deporte> deporte;

}

```
 



## @Ignore

La anotación @Ignore le indica a Jmoordbcore que descarte el atributo especificado, ya que no corresponde a un atributo de un documento en la colección.

La interface Ignore está definida de la siguiente manera:

```java

@Documented
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Ignore {
    String value() default "";
    String commentary() default "";
}


```


En el ejemplo siguiente, se define un atributo llamado resultado. Este atributo será usado para almacenar el resultado de algunas operaciones. Este atributo no será almacenado en el documento en la base de datos.


```java

@Entity
public class Operacion{
       @Id
       private String idoperacion;
       @Column
       private String detalle;
       @Column
       private Integer cantidad;
       @Column 
       private Double valor;

       @Ignore
       private Double resultado;
}


```




En el siguiente ejemplo, se implementa una clase anotada con @DocumentEmbeddable, que contiene referencias a otras colecciones. Esta clase incluye un atributo de tipo Long llamado id, que se utiliza como un identificador auxiliar para ciertas operaciones.


```java


@DocumentEmbeddable
public class Profile {

    @Ignore
    private Long id;

    @Referenced(from = "applicative", localField = "idapplicative")
    private Applicative applicative;
    @Referenced(from = "role", localField = "idrole")
    private Role role;
    @Referenced(from = "departament", localField = "iddepartament")
    private Departament departament;

    @Column
    private Boolean active;
}


```




## @Domain

Las clases declaradas con la anotación @Domain son ignoradas por Jmoordbcore. Estas clases no se almacenan en la base de datos.


La interface Domain está definida de la siguiente manera:

```java

@Documented
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface Domain {
    String commentary() default "";
   
}

```

En el siguiente ejemplo, se define una clase que no será almacenada en la base de datos:

```java

@Domain(commentary = "Se usa para calular los totales del proyecto")
public class ProyectoEstadistica {
    public Integer totalSprint;
    public Integer totalTarjetasBacklog;
    public Integer totalTarjetasPendiente;
    public Integer totalTarjetasProgreso;
    public Integer totalTarjetasFinalizado;
    public Long proyecto;

    public ProyectoEstadistica() {
    }

  //set/get
    
}


```

## @ViewEntity 

La anotación @ViewEntity permite que se defina una vista dinámica, que incluye solo ciertos atributos. Estas se usan en conjunto con la anotación @ViewReferenced, y serán explicados en el capítulo 9.




## Resumen

En este capítulo se explicaron las anotaciones que se emplean para definir entidades, a través de ejemplos que incluyen documentos embebidos y referenciados. En el siguiente capítulo, se abordará el tema de repositorios.

