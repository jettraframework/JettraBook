# Capítulo 15

Este capítulo se enfoca en la utilización de Microprofile RestClient para consumir servicios Rest, y en cómo visualizar los resultados a través de Jakarta Server Faces.

Este capítulo incluye los temas:

* Microprofile RestClient

* Consumir Endpoints

* Pruebas de Integración

* Jakarta Server Faces



## Microprofile RestClient

La especificación proporciona la capacidad de definir clientes Rest a través de interfaces, simplificando el uso y el desarrollo de aplicaciones que interactuarán con Microservicios.

Para más detalles, puede consultar la especificación en [https://github.com/eclipse/microprofile-rest-client/releases](https://github.com/eclipse/microprofile-rest-client/releases).


En este capítulo, vamos a desarrollar una aplicación Jakarta EE que implementará un cliente para comunicarnos con el Endpoint que creamos en el capítulo 11.

Inicie un proyecto utilizando Jakarta EE Starter. Visite el sitio [https://start.jakarta.ee/](https://start.jakarta.ee/) y seleccione las opciones de Jakarta EE 10, Jakarta EE Profile, Java SE Version, Runtime y Docker Support, tal como se ilustra en la figura siguiente.

![](figura_15_00.png)




Presione el botón **Generate** para descargar el archivo jakartaee-hello-world.zip que contiene el proyecto.


Después de descomprimir el archivo, abra el proyecto desde NetBeans IDE o su editor de código favorito. Haga clic derecho en el proyecto y seleccione Rename para cambiar los tres atributos del proyecto capitulo15 **Change display name, ArtifactID, Rename Folder**, tal como se muestra en la siguiente figura.


![](figura_15_01.png)


Puede observar que el proyecto se muestra con el nombre de capitulo15, tal como se muestra en la siguiente figura

![](figura_15_02.png)

Para convertir el proyecto a Payara Micro, siga los siguientes pasos:

* Haz clic derecho en el proyecto seleccionado.

* Eliga la opción New luego seleccione Other 

* En la sección Categories, seleccione  **Payara** y en File Types, elija  **Payara Micro maven plugin**

![](figura_15_03.png)



Presione el botón **Next**. Aparecerá un cuadro de diálogo que le solicitara que indique la versión de Payara Micro

![](figura_15_04.png)


Abra el archivo pom.xml y modifique las versiones de las siguientes propiedades:

```xml
<payara.version>6.2024.11</payara.version>
```

En la sección plugins

```xml
<groupId>fish.payara.maven.plugins</groupId> 
a la versión <version>2.0</version>
```

 y cambie <finalName> a  <finalName>capitulo15</finalName>.

Antes de continuar, asegúrese de probar el proyecto ejecutando

```shell

mvn clean verify payara-micro:start

```

Por favor, detenga la ejecución del proyecto.


A continuación, se mostrará cómo configurar el archivo Dockerfile

### Configurar DockerFile


Abra el archivo DockerFile y realice las siguientes modificaciones: añada payara-micro en el comando FROM y use COPY para copiar el archivo capitulo15.war, tal como se muestra a continuación

```
FROM payara/micro:6.2024.11-jdk21

COPY target/capitulo15.war $DEPLOY_DIR


```

### Archivo microprofile-config.properties

Para crear el archivo microprofile-config, siga estos pasos:

Navegue a la carpeta /src/main del proyecto capitulo15 a través de la pestaña Files. Dentro de esta, cree una nueva carpeta llamada resources. Dentro de resources, cree otra carpeta llamada 'META-INF'. Finalmente, dentro de 'META-INF', genere un archivo llamado microprofile-config.properties.



![](figura_15_05.png)

Por ahora, este archivo estará vacío, pero posteriormente añadiremos propiedades a él.


## Consumir endpoints

Como recordará del capítulo 11, generamos algunos endpoints que vamos a utilizar en este capítulo.

Antes de proceder, asegúrese de abrir y ejecutar el proyecto del capítulo 11.


```shell


mvn clean verify payara-micro:bundle


java -jar target/capitulo11-microbundle.jar --noHazelcast


```

Los endpoints que se visualizan al ejecutar el proyecto están definidos en el controlador EstudianteController.java.

```
Payara Micro URLs:
http://avbravo:8080/

'ROOT' REST Endpoints:
GET     /api/application.wadl
GET     /api/estudiante
POST    /api/estudiante
PUT     /api/estudiante
GET     /api/estudiante/findbynombre
GET     /api/estudiante/histogram
DELETE  /api/estudiante/{idestudiante}
GET     /api/estudiante/{idestudiante}
GET     /api/faulttolerance
GET     /openapi/
GET     /openapi/application.wadl

]]

```



## Pruebas de Integración

[JUnit](https://junit.org/) es un marco de trabajo que facilita la creación de pruebas para aplicaciones Java.

Usaremos JUnit para crear una prueba de integración que verifique la comunicación con los Endpoints.

Antes de iniciar, asegúrese de realizar la consulta utilizando Curl o Postman para obtener los datos de respuesta


```shell

curl --location --request GET http://localhost:8080/api/estudiante/


```


La consulta produce la siguiente respuesta:

```shell

[
  {
   "edad":49,"idestudiante":"1-2-3","nombre":"Aristides"
  },
  {
  "edad":25,"idestudiante":"7-8-5","nombre":"Maria"
  },
  {
  "edad":25,"idestudiante":"4-3-8","nombre":"Luisa"
  }
]

```

Utilizaremos esta respuesta para validar la prueba.



Siga los pasos a continuación para crear una prueba JUnit:

1. En el menú, elija la opción File luego seleccione New File. Luego, en Categories, seleccione Unit Test y en File Types, elija JUnit Test.

![](figura_15_06.png)


2. Nombre la prueba como EstudianteIT y asegúrese de que se ubique en Test Packages.

![](figura_15_07.png)

3. En el archivo pom.xml, proceda a realizar la configuración del plugin maven-failsafe.

```xml

<plugin>
<groupId>org.apache.maven.plugins</groupId>
<artifactId>maven-failsafe-plugin</artifactId>
<version>${maven-failsafe-plugin.version}</version>
 <executions>
   <execution>
      <goals>
          <goal>integration-test</goal>
          <goal>verify</goal>
      </goals>
   </execution>
 </executions>
</plugin>


```

4. A continuación, en la clase EstudianteIT, escriba el código necesario para verificar los datos que devuelve el Endpoint.

```java

public class EstudianteIT {

  HttpClient client;

  @BeforeEach
  public void setUp() {
    client = HttpClient.newHttpClient();
  }

    
 @Test
 void findAll() throws Exception {

   HttpRequest request = HttpRequest
         .newBuilder(new URI("http://localhost:8080/api/estudiante"))
         .build();
   HttpResponse<String> response = client.send(request, BodyHandlers.ofString());
   assertEquals("[
      {\"edad\":49,\"idestudiante\":\"1-2-3\",\"nombre\":\"Aristides\"},
      {\"edad\":25,\"idestudiante\":\"7-8-5\",\"nombre\":\"Maria\"},
      {\"edad\":25,\"idestudiante\":\"4-3-8\",\"nombre\":\"Luisa\"}]",
      response.body());
    }
   
}


```

5. Si el proyecto capitulo15 está en ejecución, deténgalo.

6. Ejecute el siguiente comando

```shell

mvn clean verify

```


El resultado muestra que se ha ejecutado una prueba con éxito.


![](figura_15_08.png)

```

 Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

```


Si desea omitir la ejecución de las pruebas, incluya -DskipTests, tal como se muestra en el siguiente segmento

```shell

 mvn clean verify -DskipTests

```

## Microprofile RestClient

Microprofile RestClient, que forma parte de la especificación Microprofile, facilita la comunicación con microservicios al permitir el consumo de Endpoints. Esto se logra declarando metodos con anotaciones en una interfaz, lo que simplifica la interacción del cliente con el microservicio.




Para integrar Microprofile RestClient en su proyecto, realice los siguientes pasos:
Abra el archivo pom.xml e incluya las dependencias deMicroprofile RestClient.


```xml

<microprofile.version>6.1</microprofile.version>
<microprofile-config-api.version>3.1</microprofile-config-api.version>
<version.jmoordb-core-annotations>2.0.2</version.jmoordb-core-annotations>


<dependency>
    <groupId>org.eclipse.microprofile</groupId>
    <artifactId>microprofile</artifactId>
    <version>${microprofile.version}</version>
    <type>pom</type>
    <scope>provided</scope>
</dependency>

<dependency>
    <groupId>org.eclipse.microprofile.config</groupId>
    <artifactId>microprofile-config-api</artifactId>
    <version>${microprofile-config-api.version}</version>
</dependency>

<dependency>
    <groupId>com.github.avbravo</groupId>
    <artifactId>jmoordb-core-annotations</artifactId>
    <version>${version.jmoordb-core-annotations}</version>
</dependency>



```



Debe crear una entidad Estudiante que contenga los mismos atributos que la entidad presentada en el capítulo 11.

```java

@Entity()
public class Estudiante {

    @Id
    private String idestudiante;
    @Column
    private String nombre;

    @Column
    private Integer edad;

//...

}

```

Emplee la anotación @RegisterRestClient para definir el parámetro baseUri del Endpoint. Si no se especifica el parámetro baseUri, se utilizará la configuración predeterminada del archivo microprofile-config.properties.

Como ejemplo:

```java

@RegisterRestClient(baseUri ="http://localhost:8080/api/estudiante")

o

@RegisterRestClient()

```


Cree la interfaz EstudianteRestClient.java y revise los métodos que se definieron en la clase EstudianteController.java del capítulo 11. Estos métodos servirán como base para definir los métodos de la interfaz.

Asegúrese de copiar únicamente las definiciones de los métodos en la interfaz EstudianteRestClient.java.

A continuación, se presenta el fragmento de código correspondiente a la interfaz finalizada:

```java

@RegisterRestClient()
@Path("/estudiante")
public interface EstudianteRestClient {

    @GET
    @Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})
    public List<Estudiante> findAll();

    @GET
    @Path("{idestudiante}")
    public Estudiante findByIdestudiante(@PathParam("idestudiante") String idestudiante);

    @POST
    public Response save(@RequestBody(content = @Content(mediaType = "application/json",
            schema = @Schema(implementation = Estudiante.class))) Estudiante estudiante);

    @PUT
    public Response update(@RequestBody(content = @Content(mediaType = "application/json", 
            schema = @Schema(implementation = Estudiante.class))) Estudiante estudiante);

    @DELETE
    @Path("{idestudiante}")
    public Response delete(@Parameter(required = true, example = "1",
            schema = @Schema(type = SchemaType.STRING))
            @PathParam("idestudiante") String idestudiante);

    @GET
    @Path("findbynombre")
    @Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})
    public List<Estudiante> findByNombre(@QueryParam("nombre") String nombre);

}

```


Dado que en la interfaz se ha definido un RegisterRestClient() vacío, es necesario establecer el URI en el archivo microprofile-config.properties.

Para hacer esto, especificamos el paquete que contiene la interfaz y proporcionamos la URL del endpoint, tal como se muestra a continuación:


```shell

#--------------------------------------------------------------
# Microprofile RestClient
#-------------------------------------------------------------
org.eclipse.jakarta.hello.restclient.EstudianteRestClient/mp-rest/url=http://localhost:8080/api/



```


## Jakarta Server Faces

Jakarta Server Faces es la especificación que reemplaza a Java Server Faces, proporcionando un marco de trabajo para el desarrollo de interfaces web en Java.

Hay múltiples bibliotecas que implementan esta especificación. Entre ellas, [Primefaces](http://primefaces.org) se destaca por su amplio conjunto de componentes predefinidos y listos para ser utilizados.

La versión 4.0 de Jakarta Faces facilita la creación de interfaces de manera programática.


Para utilizar Jakarta Faces y Primefaces es necesario incorporar las dependencias en el proyecto.

```xml
<properties>
  <primefaces.version>13.0.0</primefaces.version>
  <version.jmoordbutilfaces>3.4</version.jmoordbutilfaces>
  <version.jmoordb-core-annotations>2.0.2</version.jmoordb-core-annotations>
</properties>

 </dependencies>
  <dependency>
    <groupId>jakarta.faces</groupId>
    <artifactId>jakarta.faces-api</artifactId>
    <version>4.0.1</version>
    <scope>provided</scope>
   </dependency>

   <dependency>
    <groupId>org.primefaces</groupId>
    <artifactId>primefaces</artifactId>
    <version>${primefaces.version}</version>
    <classifier>jakarta</classifier>        
   </dependency>
 </dependencies>

 <repositories>
    <repository>
      <id>jitpack.io</id>
        <url>https://jitpack.io</url>
      </repository>
 </repositories>

```



Continúe con la configuración del archivo web.xml, localizado en el directorio WEB-INF. Añada el contenido siguiente para establecer el tema Saga y ajustar los parámetros correspondientes.

```xml
     
<context-param>
    <param-name>jakarta.faces.PROJECT_STAGE</param-name>
    <param-value>Development</param-value>
</context-param>

<servlet>
    <servlet-name>Faces Servlet</servlet-name>
    <servlet-class>jakarta.faces.webapp.FacesServlet</servlet-class>
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>Faces Servlet</servlet-name>
    <url-pattern>/faces/*</url-pattern>
</servlet-mapping>
<welcome-file-list>
    <welcome-file>faces/index.xhtml</welcome-file>
</welcome-file-list>


<context-param>
    <param-name>primefaces.THEME</param-name>
    <param-value>saga</param-value>
</context-param>

```


Modifique el archivo en la sección <welcome-file-list> para que apunte a index.html, tal como se ilustra a continuación.


```
<welcome-file-list>
    <welcome-file>index.xhtml</welcome-file>
</welcome-file-list>

```



El archivo beans.xml ha sido añadido al directorio WEB-INF.

```xml

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="https://jakarta.ee/xml/ns/jakartaee"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="https://jakarta.ee/xml/ns/jakartaee https://jakarta.ee/xml/ns/jakartaee/beans_3_0.xsd"
       bean-discovery-mode="all">
</beans>


```

Además el archivo faces-config.xml, que contiene la configuración para Jakarta Faces.

```xml

<?xml version="1.0" encoding="UTF-8"?>
<faces-config
    xmlns="https://jakarta.ee/xml/ns/jakartaee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="https://jakarta.ee/xml/ns/jakartaee https://jakarta.ee/xml/ns/jakartaee/web-facesconfig_4_0.xsd"
    version="4.0">

    <name>capitulo15</name>
    <application>
        
    </application>

</faces-config>


```

Genere la clase ApplicationConfig.java para establecer la configuración mediante la anotación @FacesConfig.

```java
@CustomFormAuthenticationMechanismDefinition(
        loginToContinue = @LoginToContinue(
                loginPage = "/index.xhtml",
                useForwardToLogin = false
            )
)
@FacesConfig
@ApplicationScoped
public class ApplicationConfig {
    
}

```

Estas configuraciones nos permiten integrar Jakarta Server Faces en nuestro proyecto.



### Clases Jakarta Faces

Cree una clase denominada EstudianteFaces, aplique la anotación @Named para habilitar su uso como CDI y defina su alcance con @ViewScoped. 

Luego, inyecte EstudiantRestClient, defina una lista de estudiantes y en el método init, llame al método findAll() para invocar el Endpoint que retornará todos los estudiantes.


```
@Named
@ViewScoped
public class EstudianteFaces implements Serializable {

    private static final long serialVersionUID = 1L;

    @Inject
    EstudianteRestClient estudianteRestClient;

    List<Estudiante> estudianteList = new ArrayList<>();

    public List<Estudiante> getEstudianteList() {
        return estudianteList;
    }

    public void setEstudianteList(List<Estudiante> estudianteList) {
        this.estudianteList = estudianteList;
    }

    @PostConstruct
    public void init() {
        try {
            estudianteList = estudianteRestClient.findAll();
        } catch (Exception e) {
            FacesUtil.errorMessage(FacesUtil.nameOfClassAndMethod() + " " + e.getLocalizedMessage());
        }

    }

}


```


Para crear la página index.xhtml, navegue a la carpeta WebPages, luego desde el menú de NetBeans, siga la ruta: File luego seleccione New File , luego Java Server Faces y seleccione JSF Page. Una vez creada la página, modifique su contenido de acuerdo a las instrucciones siguientes:

```xml
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:h="jakarta.faces.html"
      xmlns:p="http://primefaces.org/ui"
      lang="en"
      xmlns:f="jakarta.faces.core">
    <h:head>
        <title>MicroProfile RestClient</title>
    </h:head>
    <h:body>
       <f:view contentType="text/html">
        <p>
            <a href="https://jakarta.ee"><img width="10%" src="images/jakartaee_logo.jpg"/></a>
        </p>
        <h:form>
            <p:growl/>
            <p:dataTable var="item" value="#{estudianteFaces.estudianteList}"   paginator="true" rows="25">
                <f:facet name="header">
                    <p:outputLabel value="Listado de estudiantes"/>
                </f:facet>

                <p:column headerText="idestudiante">
                    <p:outputLabel value="#{item.idestudiante}"/>
                </p:column>

                <p:column headerText="nombre">
                    <p:outputLabel value="#{item.nombre}"/>
                </p:column>

                <p:column headerText="edad">
                    <p:outputLabel value="#{item.edad}"/>
                </p:column>

            </p:dataTable>
        </h:form>
       </f:view>
    </h:body>
</html>

```

El componente <p:datatable> de PrimeFaces genera una tabla donde los valores se definen en el atributo **value**, que está asociado a una lista de una clase Java. La palabra clave **var** se utiliza para definir una variable que permite iterar sobre los elementos de la lista. Se utiliza <p:column> para crear las columnas que se mostrarán en la tabla. Por último, <p:outputLabel> se utiliza para generar etiquetas de texto que se colocan en cada columna, representando los valores de los atributos de la entidad **estudiante**.


Inicie la ejecución del proyecto utilizando el siguiente comando:


```shell

 mvn clean verify payara-micro:start


```

Abra su navegador web y visite la siguiente URL [http://localhost:8081/capitulo15/](http://localhost:8081/capitulo15/)


![](figura_15_09.png)


Podemos visualizar los documentos almacenados en la base de datos MongoDB a través del Endpoint que consultamos utilizando RestClient.


## Resumen

Este capítulo detalla el uso de Microprofile RestClient para consumir Endpoints y la implementación de JakartaFaces para la visualización de resultados. El próximo capítulo abordará Helidon.