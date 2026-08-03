# Capítulo 21


En este capítulo, vamos a explorar la implementación de Java Records con JmoordbCore

Este capítulo incluye los temas:

* Java Records
* @Record


Pasos:

1. Crear el proyecto desde [Payara Started](https://start.payara.fish/?_gl=1*16tfx2f*_gcl_au*MTc5Mzg4ODYzOC4xNzM4MjAzNDQz*_ga*MTM5MTM1MjU5NC4xNzMwMjU0MTU1*_ga_N1E0KM13LP*MTc0MTc0OTEyMS41Mi4xLjE3NDE3NDkxNDMuMzguMC4xNjM1NTk3NzQ1)

Coloque como nombre capitulo 21

2. Descomprima el archivo

3. De permisos de ejecución


4. Ejecute

mvn clean package payara-micro:dev



## Resumen





### 1. Definir un `record`
```java
public record Persona(String nombre, int edad) {
    // Métodos "with" para crear nuevas instancias con valores actualizados
    public Persona withNombre(String nuevoNombre) {
        return new Persona(nuevoNombre, this.edad);
    }

    public Persona withEdad(int nuevaEdad) {
        return new Persona(this.nombre, nuevaEdad);
    }
}
```

### 2. Crear una instancia y "actualizar" valores
```java
public class Main {
    public static void main(String[] args) {
        // Crear una instancia inicial
        Persona persona1 = new Persona("Ana", 25);
        System.out.println("Original: " + persona1);

        // "Actualizar" el nombre creando una nueva instancia
        Persona persona2 = persona1.withNombre("Carlos");
        System.out.println("Nombre actualizado: " + persona2);

        // "Actualizar" la edad creando una nueva instancia
        Persona persona3 = persona1.withEdad(30);
        System.out.println("Edad actualizada: " + persona3);
    }
}
```

### Salida:
```
Original: Persona[nombre=Ana, edad=25]
Nombre actualizado: Persona[nombre=Carlos, edad=25]
Edad actualizada: Persona[nombre=Ana, edad=30]
```

### Explicación:
- **Inmutabilidad**: Los `record` no permiten modificar campos directamente. En su lugar, se crean nuevas instancias con los valores actualizados.
- **Métodos `with`**: Son métodos personalizados que facilitan la creación de nuevas instancias con un campo modificado.
- **Alternativa**: También puedes usar el constructor directamente:
  ```java
  Persona persona4 = new Persona(persona1.nombre(), 35);
  ```

Este enfoque garantiza que los datos sean consistentes y seguros en entornos concurrentes.

