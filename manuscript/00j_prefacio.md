# Prefacio




El desarrollo de aplicaciones Java, especialmente en entornos empresariales, ha ganado relevancia en los últimos años. La aparición de bases de datos NoSQL, el uso de contenedores como Docker y la adopción de nuevos modelos de desarrollo han creado una necesidad de aprendizaje para implementar soluciones efectivas.


Este libro se centra en el desarrollo de aplicaciones empresariales Jakarta EE con bases de datos NoSQL (MongoDB), utilizando el marco de trabajo Java **Jmoordb-core**.


Este es un libro práctico que te permitirá conocer las especificaciones Jakarta EE y Eclipse Microprofile mientras trabajas en proyectos reales con productos tales como Payara Micro, OpenLiberty y Helidon.


Este libro está diseñado para desarrolladores principiantes que están comenzando a trabajar con NoSQL y Microservicios, proporciona una gran cantidad de referencias, códigos de ejemplo y consejos útiles sobre temas generales.



**Quién puede leer este libro**  



Si no posee conocimientos del lenguaje de programación Java, este libro es una guía sencilla y práctica. Cada capítulo cuenta con ejemplos prácticos diseñados para enseñar de manera similar a las recetas de cocina.

Si usted es un desarrollador Java con conocimientos básicos de Jakarta EE, NoSQL, y Microprofile y está interesado en crear microservicios en Java, este libro muestra como desarrollar aplicaciones de manera eficiente y rápida.


**Que abarca el libro**  


Capítulo 1, Introduce a Jakarta EE, Eclipse Microprofile y las bases de datos NoSQL.
 
Capítulo 2, Presenta una introducción a Jmoordbcore, destacando sus  aspectos relevantes y características. Le permite familizarse con la arquitectura de Jmoordbcore para aprovechar el máximo provecho al marco de trabajo.

Capítulo 3, Explica cómo definir entidades con sus correspondientes anotaciones. Le permitirá declarar documentos embebidos y crear referencias entre uno o varios documentos desde su código Java.

Capítulo 4, Aprenderá a definir repositorios utilizando anotaciones que definen las operaciones sobre colecciones y documentos de la base de datos.


Capítulo 5, Aborda la limitación de campos autoincrementables en bases de datos NoSQL. Este capítulo le ayudará a comprender cómo Jmoordbcore implementa datos secuenciales para resolver esta limitación. 


Capítulo 6, Explica cómo usar Jakarta RESTful Web Services para manejar fechas, utilizando Jmoordbcore.


Capítulo 7, Conocerá el driver oficial de Java para MongoDB, también se explica el código generado por el marco de trabajo Jmoordbcore.


Capítulo 8, Aprenderá a crear referencias entre documentos, emplear documentos embebidos y realizar paginación y ordenación.


Capítulo 9, Explica cómo se definen las vistas dinámicas utilizando la anotación @ViewEntity, lo que mejora el rendimiento de las consultas a la base de datos.


Capítulo 10, Se centra en MongoDB Atlas, la base de datos en la nube, enseñándole la administración básica de la misma.


Capítulo 11, Muestra cómo implementar métricas en las aplicaciones utilizando Microprofile Metrics.


Capítulo 12, Introduce Prometheus/Grafana, herramientas facilitan el monitoreo de los microservicios.


Capítulo 13, Le enseña a usar JMeter para ejecutar pruebas y métricas sobre la aplicación.


Capítulo 14, Le muestra cómo usar Docker de manera sencilla, generar imágenes y subirlas a Docker Hub.


Capítulo 15, Explica cómo emplear Microprofile RestClient para consumir servicios rest de forma nativa y simple. Incluye un ejemplo usando Jakarta Server Faces para generar la aplicación Web.


Capítulo 16, Aprenderá a usar Helidon, un marco de trabajo muy ligero para la creación de microservicios.


Capítulo 17, Presenta OpenLiberty, la apuesta de IBM para la creación de microservicios ligeros. Este capítulo le enseña a crear un proyecto con Jmoordbcore.


Capítulo 18, Explica brevemente las especificaciones Jakarta NoSQL, Jakarta Data y su integración en Jmoordbcore.

Capítulo 19, Aprenderá a definir bases de datos y colecciones dinámicas.

Capítulo 20, Explora el uso de campos llaves de tipo ObjectId y UUID.

Capítulo 21, Muestra la implementación de JMoordbCore con JettraFramework.





**Código fuente**  




Todos los ejemplos presentados en este libro se encuentran disponibles en el repositorio [https://github.com/avbravo/jmoordb-corebookexamples.git](https://github.com/avbravo/jmoordb-corebookexamples.git).




Las actualizaciones más recientes del marco de trabajo se estarán publicando en el sitio [https://avbravo.github.io](https://avbravo.github.io).




**Convenciones** 
 
Este libro utiliza diversos estilos de texto para facilitar la distinción entre las secciones y elementos que lo componen.


Las palabras reservadas serán escritas en **negrita**, mientras que los segmentos de código se presentarán de la siguiente apariencia:


```java


public class Prueba{  
   void save( ) {

   }
}


```


* Los enlaces se resaltarán en color azul [https://avbravo.blogspot.com](https://avbravo.blogspot.com)


* Los términos **entity** y **entidad**, se usarán para referirse a clases Java que representan un documento en la base de datos.


* Los términos **repository** y **repositorio** se usarán para referirse a las interfaces para generar operaciones a la base de datos.


* Encontrara algunos terminos en Ingles para hacer referencia directa a las especificaciones o herramientas utilizadas.

**Importante**


* Es un libro práctico, sencillo, con muchos ejemplos, orientado a personas que inician en Java.


* En algunos capítulos no se incluyen las **pruebas** (test). Esto se ha hecho intencionalmente para que el lector pueda practicar realizando sus propias implementaciones.


* Recomendamos utilizar la versión más reciente de Java, que es la 21 al momento de escribir este libro. Sin embargo, usted puede utilizar cualquier versión a partir de la 11 en adelante.

* Aunque se utiliza Apache NetBeans IDE para los ejercicios en este libro, no es un requisito.



**Errata**  


Si encuentra algún error en el libro, agradeceríamos recibir sus comentarios al respecto. 


**Preguntas**   


Si tiene alguna pregunta, por favor escriba un correo electrónico a [avbravo@gmail.com](avbravo@gmail.com)













