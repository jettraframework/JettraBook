# Capítulo 23


En este capítulo, vamos a explorar la implementación de Generadores de codigo incluido en Jmoordbcore

Revise los ejemplos en el proyecto de ejemplo capitulo23.

Este capítulo incluye los temas:


## Generadores Patrón Builder
Pasos:

* @CoreBuilder : Aplica a nivel de clases Java o Java Records
* @CoreBuilderProperty: Aplica a nivel de metodos set.
* @ViewModel: Aplica para generar una clase ViewModel que se usara con el patron MVC, en el front-end.



### @CoreBuilder

Permite generar una clase que implementa el patrón Builder y agiliza la creación de instancias de objetos.




**Ejemplo**

En el siguiente ejemplo se genera el Builder para una entidad 

```java

@CoreBuilder
public class Car {

    private String name;
    private int age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "Car{" + "name='" + name + '\'' + ", age=" + age + '}';
    }

}


```

Al compilar el proyecto se genera de manera automática el siguiente código:

```java
public class CarBuilder {
    private final Car target = new Car();

    public static CarBuilder builder() {
        return new CarBuilder();
    }
    public CarBuilder withName(java.lang.String name) {
        target.setName(name);
        return this;
    }
    public CarBuilder withAge(int age) {
        target.setAge(age);
        return this;
    }
    public Car build() {
        return target;
    }
}


```

Para usarlo en una clase

```java
 Car c = new CarBuilder()
        .withName("Toyota")
        .build();
```


**Ejemplo 2**

Se aplica a Java Records de igual manera

```java
@CoreBuilder
public record Celular(Long id, String name) {
    
    
}
```

Genera el codigo

```java
public final class CelularBuilder {

  java.lang.Long id;        
  java.lang.String name;        

  public CelularBuilder(){
  }

/**
Constructor
**/
 public CelularBuilder(
         java.lang.Long id  ,   
         java.lang.String name     
  ){
   this.id = id;
   this.name = name;
 }

/**
Fields
**/
public CelularBuilder id(java.lang.Long id) {
        this.id = id;
        return this;
}
public CelularBuilder name(java.lang.String name) {
        this.name = name;
        return this;
}

public Celular build() {
        return new Celular(
                   id ,
                   name 
 );
}

}


```


Se puede utilizar de la siguiente manera

```java
 Celular celular = new CelularBuilder()
                .id(7L)
                .name("Aristides")
                .build();
```


### @CoreBuilderProperty

Permite generar una clase Builder que implementa el patrón Builder definiendo solamente a nivel de métodos.
No se aplica a **Java Records**.



**Ejemplo**

Se aplica a nivel de campo, y genera el Builder para campos específicos. No es necesario definirlo a  nivel de clase.

```java
@Entity
public class Deporte {

    @Id
    private Long iddeporte;
    @Column
    private String name;
    @Column
    private String grupo;

    public Deporte() {
    }

    public Deporte(Long iddeporte, String name) {
        this.iddeporte = iddeporte;
        this.name = name;
    }

    public Long getIddeporte() {
        return iddeporte;
    }

    @CoreBuilderProperty
    public void setIddeporte(Long iddeporte) {
        this.iddeporte = iddeporte;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getGrupo() {
        return grupo;
    }

    @CoreBuilderProperty
    public void setGrupo(String grupo) {
        this.grupo = grupo;
    }

}

```


Al compilarse genera

```java
public class DeporteBuilder {
    private final Deporte target = new Deporte();

    public static DeporteBuilder builder() {
        return new DeporteBuilder();
    }
    public DeporteBuilder setIddeporte(java.lang.Long value) {
        target.setIddeporte(value);
        return this;
    }
    public Deporte build() {
        return target;
    }
}


```


Ejemplo de uso

```java

    Deporte d = new DeporteBuilder()
                    .setIddeporte(6L)
                    .build();

```


## @ViewModel



```java
@ViewModel
public record Celular(Long id, String name) {  
    
}

```


Genera el siguiente código:


```java
public class CelularViewModel {

    java.lang.Long id;
    java.lang.String name;

    public CelularViewModel() {
    }

    /**
     * Constructor
*
     */
    public CelularViewModel(
            java.lang.Long id,
            java.lang.String name
    ) {
        this.id = id;
        this.name = name;
    }

    /**
     * Fields
*
     */
    public void setId(java.lang.Long id) {
        this.id = id;
    }

    public java.lang.Long getId() {
        return id;
    }

    public void setName(java.lang.String name) {
        this.name = name;
    }

    public java.lang.String getName() {
        return name;
    }

    /**
     * ToString
*
     */
    @Override
    public String toString() {

        return "CelularViewModel{"
                + "id=" + id + ","
                + "name=" + name + ""
                + '}';

    }

    @Override
    public int hashCode() {
        int hash = 7;
        hash = 97 * hash + Objects.hashCode(this.id);
        hash = 97 * hash + Objects.hashCode(this.name);
        return hash;
    }

    public Celular toRecord() {
        return new Celular(
                id,
                name
        );
    }

    public Celular toRecord(CelularViewModel... vm) {
        if (vm.length != 0) {
            return new Celular(
                    vm[0].id,
                    vm[0].name
            );
        } else {
            return new Celular(
                    id,
                    name
            );
        }
    }

    public CelularViewModel toViewModel(Celular celular) {
        CelularViewModel result = new CelularViewModel();
        try {
            result.setId(celular.id());
            result.setName(celular.name());
        } catch (Exception e) {
            System.out.println("toViewModel() " + e.getLocalizedMessage());
        }
        return result;
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
        final CelularViewModel other = (CelularViewModel) obj;
        if (!Objects.equals(this.id, other.id)) {
            return false;
        }
        if (!Objects.equals(this.name, other.name)) {
            return false;
        }
        return true;
    }

}

```

Se puede utilizar de la siguiente manera

```java
  /**
         * ViewModel Mediante atributos
         */
        CelularViewModel celularAttributes = new CelularViewModel();
        celularAttributes.setId(4L);
        celularAttributes.setName("Ana");
```


## @ViewModelBuilder

Permite crear un Builder a partir para ser usado con un ViewModel, generado a partir de un Java Record.

```java
@ViewModelBuilder
public record Celular(Long id, String name) {
    
    
}
```

Genera el código

```java
public class CelularViewModelBuilder {
    private CelularViewModel target = new CelularViewModel();

    public static CelularViewModelBuilder builder() {
        return new CelularViewModelBuilder();
    }
    public CelularViewModelBuilder withId(java.lang.Long id) {
        target.setId(id);
        return this;
    }
    public CelularViewModelBuilder withName(java.lang.String name) {
        target.setName(name);
        return this;
    }
    public CelularViewModel build() {
        return target;
    }
}

```

Se puede utilizar de la siguiente manera

```java
  // View Model usando Builder
        CelularViewModel cel2 = new CelularViewModelBuilder()
                .withId(4L)
                .withName("Dayana")
                .build();
```



## Java Record combinado

Es posible usar todas las anotaciones con un Java Record

```java
@CoreBuilder
@ViewModel
@ViewModelBuilder
public record Celular(Long id, String name) {
    
    
}
```


## Resumen

Este capitulo se explico como puedes generar clases que implementan el patrón Builder que facilitan la creación de objetos.