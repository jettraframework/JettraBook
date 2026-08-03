# Capítulo 09

Este capítulo se centra en el uso de la anotación @ViewEntity para crear vistas dinámicas que extraen partes específicas de un documento, especialmente útil cuando se manejan múltiples referencias anidadas.



Este capítulo incluye los temas:

* Introducción

* Uso de @ViewEntity

* Uso de @ViewReferenced


## Introducción
 
La anotación ViewEntity facilita la creación de vistas de las colecciones, las cuales pueden reemplazar las referencias entre colecciones, optimizando la cantidad de operaciones en las referencias internas de cada entidad definida.

Estas vistas pueden ser incorporadas dentro de otra entidad, documento embebido o vista utilizando la anotación @ViewEntity y se vinculan a través de la anotación @ViewReferenced.

En Jmoordbcore, las vistas permiten la ejecución de operaciones de creación, actualización, eliminación y lectura.



Por ejemplo, proceda a definir la entidad Persona:

```java

@Entity()
public class Persona{

    @Id(strategy = GenerationType.AUTO)
    private Long idpersona;
    @Column
    private String nombre;
    @Referenced(from = "departamento", localField = "iddepartamento")
    private Departamento departamento;
    @Referenced(from = "deporte", localField = "iddeporte")
    private List<Deportes> deportes;

//set/get
}

```

A continuación, defina la entidad Deporte:


```java
@Entity()
public class Deporte{

    @Id(strategy = GenerationType.AUTO)
    private Long iddeporte;
    @Column
    private String deporte;
    
//set/get
}



```



La entidad Departamento, que se detalla a continuación, mantiene una referencia con la entidad Sede:



```java

@Entity()
public class Departamento{

    @Id(strategy = GenerationType.AUTO)
    private Long iddepartamento;
    @Column
    private String departamento;
    @Referenced(from = "sede", localField = "idsede")
    private Sede sede;

//set/get

}


```
La entidad Sede, mantiene una referencia con la entidad Institucion:


```java

@Entity()
public class Sede{

    @Id(strategy = GenerationType.AUTO)
    private Long idsede;
    @Column
    private String sede;
    @Referenced(from = "institucion", localField = "idinstitucion")
    private Institucion institucion;

//set/get
}

```


A continuación, defina la entidad Institucion:

```java

@Entity()
public class Institucion{

    @Id(strategy = GenerationType.AUTO)
    private Long idinstitucion;
    @Column
    private String institucion;
    
//set/get
}

```


Al realizar una búsqueda de Persona, el sistema primero buscará en Departamento, luego en Sede y finalmente en Institución, tal como se ilustra en la figura siguiente:

![](figura_09_00.png)



## Uso de @ViewEntity

Las vistas se definen mediante la anotación @ViewEntity, la cual es muy similar a la anotación @Entity discutida en capítulos anteriores. La principal diferencia radica en que @ViewEntity representa ciertos elementos del documento.

Por favor, proceda a crear la clase DepartamentoView, la cual servirá como representación de la vista para el departamento:

```java

@ViewEntity(collection = "departamento")
public class DepartamentoView {

    @Id(strategy = GenerationType.AUTO)
    private Long iddepartamento;
    @Column
    private String departamento;

//set/get
}


```

La clase DepartamentoView se limita a devolver el iddepartamento y el nombre del departamento.

## Uso de @ViewReferenced

La anotación @ViewReferenced se utiliza para definir referencias a una vista. Esta anotación se puede aplicar a las clases definidas con las anotaciones @Entity y @DocumentEmbedded.


Proceda a modificar la clase Persona e implemente la anotación @ViewReferenced. Observará que el atributo 'from="departamento"' se refiere a la misma colección que la anotación @Referenced:

```java

@Entity()
public class Persona{

    @Id(strategy = GenerationType.AUTO)
    private Long idpersona;
    @Column
    private String nombre;
    @ViewReferenced(from = "departamento", localField = "iddepartamento")
    private DepartamentoView departamentoView;
    @Referenced(from = "deporte", localField = "iddeporte")
    private List<Deportes> deportes;

//set/get
}

```
Ahora, proceda a declarar el repositorio correspondiente a la clase DepartamentoView:

```java

@Repository(entity = DepartamentoView.class,collection = "departament")
public interface DepartamentoViewRepository extends CrudRepository<DepartamentoView, Long> {
 
}

```


Al definirlo con la anotación @ViewReferenced, cuando se realice una consulta sobre una persona, la búsqueda se limitará únicamente a la colección Departamento. Las búsquedas adicionales a Sede e Institución no se llevarán a cabo. Este método mejora el rendimiento de la consulta, como se puede apreciar en la siguiente figura:

![](figura_09_01.png)
  

## Resumen. 

En este capítulo, se introdujo el concepto de **@ViewEntity**, que mejora los tiempos de respuesta y evita la necesidad de referencias internas en las clases. En el próximo capítulo exploraremos MongoDB Atlas.