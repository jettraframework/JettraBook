# Capítulo 10

Este capítulo se enfoca en MongoDB Atlas, el servicio de base de datos en la nube de MongoDB, y su integración con Jmoordbcore. 

Este capítulo incluye los temas:

* MongoDB Atlas

* MongoDB Atlas con Jmoordbcore


## MongoDB Atlas

MongoDB Atlas es un servicio de base de datos en la nube proporcionado por MongoDB. Es escalable y compatible con las principales plataformas de la nube, como Amazon, Azure y Google Cloud.


En este capítulo, le guiaremos en el proceso de crear una cuenta en MongoDB Atlas. Para empezar, visite el sitio [https://www.mongodb.com/atlas/database](https://www.mongodb.com/atlas/database).


Una vez que se encuentre en la página principal, que se muestra en la siguiente figura, seleccione la opción **Try Free** para iniciar el proceso de crear un registro.

![](figura_10_00.png)



A continuación, proporcione sus datos para el registro o puede regístrarse utilizando su cuenta de Google: 

![](figura_10_01.png)

Una vez finalizado el registro, recibirá una notificación en su correo electrónico para verificar sus datos:  

![](figura_10_02.png)






Por favor, haga clic en el enlace que ha recibido para comenzar el proceso de confirmación de su cuenta. Luego, presione el botón **Continue**. Esto le redirigirá a una página donde se le solicitará que ingrese sus credenciales


![](figura_10_03.png)

Una vez que haya ingresado, verá una advertencia que indica **Current IP Address not added. You will not be able to connect to databases from this address**. Haga clic en el botón **Add current IP Address**:

![](figura_10_04.png)



En el menú izquierdo, seleccione  **Network Access** bajo la sección **Security**. Aquí se muestra su dirección IP y el estado. 

![](figura_10_05.png)

Si desea agregar una nueva dirección IP, haga clic en el botón **add ip adresss.**.

Si desea tener acceso desde cualquier ubicación, ingrese la dirección IP: 0.0.0.0/0

![](figura_10_06.png)

En la sección DataServices, seleccione la opción Create a deployment 

![](figura_10_07.png)

Elija la opción **M0 FREE**  y luego haga clic en el botón Create

![](figura_10_08.png)

Se genera automáticamente un nombre de usuario y una contraseña. Si lo desea, puede modificar estos detalles, haciendo clic en el botón **Create User**

![](figura_10_09.png)

Puede especificar las direcciones IP que tendrán acceso a la base de datos, ya sean en un entorno local o en la nube:

![](figura_10_10.png)

En la parte Inferior, haga clic en el botón Finish and Close.

Con este paso, se ha creado el cluster. Ahora puede agregar datos o importar una base de datos.

En este ejemplo, utilizaremos la base de datos de ejemplo. Para hacerlo, haga clic en Import Data. Tenga en cuenta que este proceso puede tardar minutos.

Después, haga clic en el botón  Add Data:

![](figura_10_11.png)

A continuación, se presentarán las siguientes opciones

![](figura_10_12.png)


Seleccione la opción Create Database on Atlas y haga clic en el botón Start.


Introduzca 'ejemplodb' como el nombre de la base de datos, 'persona' como el nombre de la colección, y un documento en formato JSON con los valores que se indican a continuación:

```json
{
  "nombre": "Aristides",
  "email": "avbravo@gmail.com",
    "deportes": [
        { 
            "nombre":"Baloncesto"
            
        }
  ]
}

```

Presione el botón Create Database para iniciar la creación de la base de datos

![](figura_10_13.png)


Una vez que el proceso haya finalizado, podrá observar la base de datos ejemplodb, así como otras bases de datos que fueron cargadas en los pasos anteriores.

![](figura_10_14.png)


El siguiente paso consiste en verificar las conexiones a MongoDB Atlas, para ello haga clic en Data Services y posteriormente en el botón Connect.

![](figura_10_15.png)

Para conectarnos a la base de datos, utilizaremos dos métodos: uno a través de Compass y el otro mediante el driver Java integrado con Jmoordbcore

![](figura_10_16.png)


## Conexión con Java

Seleccione Driver , luego elija **Java**  asegúrese de que la versión sea: **4.3 or later**

En la parte inferior, podrá ver la conexión **mongodb+srv** que utilizaremos con Jmoordbcore.

```java
mongodb+srv://jmoordbcore:<password>@cluster0.ct56yxp.mongodb.net/?retryWrites=true&w=majority

```


Debe sustituir <password> con la contraseña del usuario de la base de datos

![](figura_10_17.png)


Si no tiene Compass instalado, elija su sistema operativo para descargar la última versión disponible. En la parte inferior de la pantalla se proporciona información sobre la conexión a Compass.

```java

mongodb+srv://jmoordbcore:<password>@cluster0.ct56yxp.mongodb.net/

```

![](figura_10_18.png)


Ahora proceda a instalar Compass en su computador. Una vez que la instalación esté finalizada, estableceremos una conexión con MongoDB Atlas.

Al iniciar, seleccione Advanced Connnections Options e inserte la URL de la conexión que generó en MongoDB Atlas. No olvide reemplazar  <password> con la contraseña del usuario de la base de datos.

Después, haga clic en Save and Connect.

![](figura_10_19.png)



Aparecerá un cuadro de diálogo que le permitirá seleccionar un nombre para la conexión y asignarle colores. Una vez realizado esto, haga clic en el botón Save and Connect

![](figura_10_20.png)

Se presentan las diversas bases de datos disponibles en MongoDB Atlas. Por favor, seleccione ejemplodb y la colección persona. Podrá ver el documento que creamos previamente

![](figura_10_21.png)


## MongoDB Atlas con Jmoordbcore

Se requiere que cree un nuevo proyecto denominado capitulo10. Este proyecto debe utilizar Jakarta EE, Eclipse Microprofile y Payara Micro, tal como se hizo en los capítulos anteriores.

Cree un paquete model y defina una entidad llamada Persona, así como un documento embebido llamado Deportes.

A continuación, se presenta un segmento de la entidad Persona:

```java

@Entity()
public class Persona {

    @Id
    private String email;
    @Column
    private String nombre;


    @Embedded
    List<Deportes> deportes;

//
}
```


A continuación, defina el documento embebido Deportes de la siguiente manera:
```java

@DocumentEmbeddable()
public class Deportes {
    @Column
    private String nombre;
//
}
```

Posteriormente, defina el repositorio para la entidad Persona siguiendo una estructura similar a la que se muestra a continuación:

```java

@Repository(entity=Persona.class)
public interface PersonaRepository extends CrudRepository<Persona, String>{
    
}


```

Para hacer accesibles los recursos, proceda a crear el controlador PersonaController:

```java

@Path("persona")
public class PersonaController {

    @Inject
    PersonaRepository personaRepository;

    @GET
    @jakarta.ws.rs.Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})
    public List<Persona> findAll() {
        return personaRepository.findAll();
    }

}

```

Como recordará, MongoDB Atlas proporciona una cadena de conexión para la base de datos. Esta cadena debe ser agregada al archivo microprofile-config.properties:

```
mongodb+srv://jmoordbcore:<password>@cluster0.ct56yxp.mongodb.net/?retryWrites=true&w=majority

```
Reemplace <password> con su contraseña de conexión a la base de datos:

![](figura_10_22.png)


Genere el archivo UberJar ejecutando los comandos apropiados de Maven en la consola:



```shell

mvn clean verify payara-micro:bundle

```



Ejecute el proyecto utilizando los comandos Maven correspondientes en la consola:

```
java -jar target/capitulo10-microbundle.jar  --noHazelcast   


```

Como alternativa, puede ejecutar el proyecto utilizando el siguiente comando:

```shell

mvn payara-micro:start

```


### Consultar con Postman

Inicie Postman y cree una nueva petición a: [http://localhost:8080/api/persona](http://localhost:8080/api/persona).

Como respuesta, se obtendrá los documentos almacenados en la colección persona de la base de datos ejemplodb en MongoDB Atlas.

## Resumen

En este capítulo, hemos explicado cómo utilizar MongoDB Atlas como servicio y crear una aplicación que interactúe con la base de datos.

En el próximo capítulo, exploraremos el uso de las APIs Microprofile Metrics para realizar mediciones y verificar del estado de los microservicios.



