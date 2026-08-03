# Capítulo 13

Este capítulo, aprenderá a instalar y configurar JMeter para realizar pruebas de rendimiento y monitorear el comportamiento de las aplicaciones.

Este capítulo incluye los temas:

* Introducción a JMeter

* Proceso de instalación

* Creación y configuración de un Thread Group

* Creación de HTTP Request

* Configuración de un Listener

* Configuración de Response Assertion

* Configuración de Duration Assertion

* Manejo de Errores

* Uso del Plugin de Maven 


## Introducción a JMeter

![](figura_13_00.png)

[Apache JMeter™](https://jmeter.apache.org/), es un software de código abierto escrito en Java, diseñado para realizar pruebas de rendimiento. Aunque inicialmente se creó para probar aplicaciones Web, en la actualidad tiene variedad de aplicaciones.

Para trabajar este capítulo, es requisito que usted ejecute el proyecto desarrollado en el capítulo 11. A continuación se muestra cómo crear un archivo UberJar y cómo ejecutar el proyecto:

```java

mvn clean verify payara-micro:bundle

java -jar target/capitulo11-microbundle.jar --noHazelcast

```



## Proceso de instalación

Para instalar JMeter, siga los siguientes pasos:

1. Descargue JMeter desde [https://jmeter.apache.org/download_jmeter.cgi](https://jmeter.apache.org/download_jmeter.cgi).

2. Verifique que tiene instalada la versión 8 o superior de la máquina virtual de Java. Recomendamos usar Java 17.

3. Una vez descargado el archivo comprimido (.zip), proceda a descomprimirlo.

4. Ingrese a la carpeta **bin** dentro de la carpeta descomprimida.

5. Ejecute el siguiente comando:


```shell

./jmeter

```

## Creación y configuración de un Thread Group

Un Thread Group permite definir el número de usuarios simultáneos que ejecutarán las pruebas y el tiempo de ejecución de las mismas.

Para crear un Thread Group, realice los siguientes pasos:

1. Inicie JMeter y seleccione Test Plan.

2. A continuación, seleccione Add Treads(Users), y luego Thread Group, tal como se muestra en la figura a continuación:

![](figura_13_01.png)

3. Establezca el número de usuarios y el tiempo en segundos. En este ejemplo, ajustamos Number of Threads (users) a 50.

![](figura_13_02.png)

Finalmente, haga clic en el botón Save. JMeter sugerirá el nombre Thread Group.jmx, usted puede modificarlo a su conveniencia.



## Creación de HTTP Request

Un HTTP Request permite realizar peticiones HTTP a un servidor.

Para crear un HTTP Request, siga los siguientes pasos:

1. De un clic derecho en el Thread Group que acaba de crear y seleccione Add. Luego, seleccione  Sampler, y despúes HTTP Request, como se muestra en la siguiente figura:

![](figura_13_03.png)

2. Introduzca los siguientes valores:

```
Server Name o IP: localhost
Port Number: 8080
HTTP Request: GET
Path: /api/estudiante/findbynombre
, luego seleccione 
```

3. Añada el parámetro nombre con el valor Maria.

La siguiente figura muestra cómo debería verse la pantalla con los valores ya establecidos:

![](figura_13_04.png)

4. Finalmente, haga clic en el botón Save para guardar los cambios.


## Configuración de un Listener

Un Listener se utiliza para escuchar las pruebas e interpretar los resultados.

Para crear un Listener, siga los siguientes pasos: 

1. Haga clic derecho en HTTP Request, luego seleccione Add, después Listener y finalmente View Results Tree, como se muestra en la siguiente figura:

![](figura_13_05.png)

2. De esta manera, se añadirá el componente View Result Tree a la estructura de elementos, como se muestra a continuación:
, para validar que la respuesta
![](figura_13_06.png)


## Configuración de Response Assertion

Response Assertion se utilizan para comprobar que la respuesta obtenida coincide con la respuesta esperada.

Para configurar un Response Assertion, siga los siguientes pasos:

1. Seleccione HTTP Request

2. Luego, elija Add

3. Despúes, Assertions 

4. Finalmente, seleccione Response Assertion.

La siguiente figura muestra el proceso:

![](figura_13_07.png)

Para realizar la consulta, ejecute el siguiente comando:

```shell
curl --location --request GET http://localhost:8080/api/estudiante/findbynombre?nombre=Maria

```

La respuesta generada por el endpoint es:

```json

[{"edad":25,"idestudiante":"7-8-5","nombre":"Maria"}]

```

Por lo tanto, para comparar la respuesta, agregela a Patterns to Test. Como se muestra en la figura a continuación:

![](figura_13_08.png)



## Configuración de Duration Assertion

Duration Assertion define el tiempo mínimo requerido para recibir una respuesta.

Para configurar un Duration Assertion, siga los siguientes pasos:

1. Seleccione HTTP Request

2. Luego, seleccione Add

3. Despúes Assertions

4. Finalmente, seleccione Duration Assertion.

La siguiente figura muestra el proceso:

![](figura_13_09.png)

En el ejemplo, hemos establecido 50 ms como el tiempo mínimo para recibir una respuesta. Sin embargo, puede ajustar los valores que considere necesarios.

![](figura_13_10.png)

Siguiendo estos pasos, hemos configurado un entorno de pruebas básico. Ahora proceda a ejecutarlo, como se muestra en la siguiente figura

![](figura_13_11.png)

Haga clic en en el botón HTTP Request de la barra para ejecutar la prueba.

Una vez que la prueba haya finalizado, seleccione ce View Result Test para visualizar los resultados.

![](figura_13_12.png)


## Manejo de Errores

Para hacer pruebas de comportamiento de las pruebas, seleccione Thread Group y cambie el número de Threads a 500, el Ram up period (seconds ) a 1 y el Loop Count a 25


![](figura_13_13.png)

Al ejecutar la prueba, observará que se generan errores durante las operaciones.

![](figura_13_14.png)


Después de esta prueba, recuerde volver a cambiar la configuración a los valores iniciales 

![](figura_13_15.png)

**Guardar el plan**

Desde el menú File, seleccione Save Test Plan As e indique la carpeta donde desea almacenar el plan. Asigne el nombre  capitulo13.jmx, como se muestra en la figura a continuación

![](figura_13_16.png)


## Uso del Plugin de Maven 

Ahora, vamos a crear un proyecto Maven que ejecutará nuestras pruebas de JMeter.

Para este ejemplo, utilizaremos NetBeans IDE para generar el proyecto Maven, pero usted puede utilizar el IDE de su preferencia.

Para comenzar. seleccione New Project desde el menú File.
Luego en Categories, seleccione Java with Maven,  y en Project, seleccione Java Application.

![](figura_13_17.png)

Haga clic en el botón Next e indique capitulo13 como nombre del proyecto. Tenga en cuenta que solo generamos una configuración básica. Para referencias y ejemplos más completos, consulte los enlaces proporcionados al inicio del capítulo. Ahora, abra el archivo pom.xml y añada la configuración del plugin de JMeter:

```xml

<build>
 <plugins>


   <plugin>
    <groupId>com.lazerycode.jmeter</groupId>
    <artifactId>jmeter-maven-plugin</artifactId>
     <version>3.8.0</version>
    <executions>
        <execution>
            <id>configuration</id>
            <goals>
                <goal>configure</goal>
            </goals>
        </execution>
        <execution>
            <id>jmeter-tests</id>
            <goals>
                <goal>jmeter</goal>
            </goals>
        </execution>
        <execution>
            <id>jmeter-check-results</id>
            <goals>
                <goal>results</goal>
            </goals>
        </execution>
    </executions>


     </plugin>
  </plugins>
</build>

```

Agregue la dependencia de JUnit a su proyecto 

```xml

<dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.13.2</version>
    </dependency>
</dependencies>

```

En la pestaña Files del IDE, nos ubicamos en src. Luego, haga clic en New, y seleccione Folder.


![](figura_13_18.png)

A continuación, ingrese /test/jmeter como el nombre del directorio, tal como se muestra en la siguiente figura


![](figura_13_19.png)


Copie el archivo capitulo13.jmx de su plan de pruebas al nuevo directorio. El resultado debería verse como se muestra en la siguiente figura

![](figura_13_20.png)


Ahora, ejecute el proyecto del capitulo11, que contiene los endpoints que vamos a probar


```java


mvn clean verify payara-micro:bundle


java -jar target/capitulo11-microbundle.jar --noHazelcast


```

**Ejecución de Pruebas**

Para ejecutar las pruebas, diríjase a la carpeta del proyecto capitulo13, ingrese el comando:


```shell

mvn clean verify


```

El plan de pruebas comenzará y, una vez finalizado, generará los resultados

![](figura_13_21.png)


## Resumen


 En este capítulo, aprendió cómo configurar JMeter, y cómo utilizar el plugin de Maven para integrar pruebas en los proyectos Java. En el siguiente capítulo, abordaremos Docker.






