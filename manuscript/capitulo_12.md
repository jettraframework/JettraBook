# Capítulo 12

Este capítulo se enfoca en la utilización de herramientas de monitoreo para evaluar las métricas en microservicios, destacando el uso de Prometheus y Grafana.




Este capítulo incluye los temas:

* Prometheus

* Grafana





## Prometheus

Prometheus es un software diseñado para monitorear métricas y generación de alertas. Generalmente, se utiliza en conjunto con Grafana.  Es recomendable la lectura de la documentación oficial de [Prometheus](https://prometheus.io/docs/prometheus/latest/getting_started/).



En este capítulo, se muestran dos formas de utilizar Prometheus: a través de la descarga directa de la aplicación o mediante Docker.


### Instalación de la Aplicación

Descargue el instalador desde la página de [https://prometheus.io/download/](https://prometheus.io/download/). Al momento de escribir este libro, la versión disponible es 2.46.0. Asegúrese de seleccionar la versión que corresponda a su sistema operativo.


![](figura_12_00.png)

Una vez descargado el archivo, proceda a descomprimirlo y abra el archivo prometheus.yml

![](figura_12_01.png)


Dentro del archivo, busque la línea que contiene 'scrape_configs'. Debera insertar un nuevo 'job_name' llamado 'capitulo12', el cual deseamos que se ejecute cada 15 segundos. En la sección 'static_config' seleccione la URL en la sección 'target', como se muestra a continuación:

```yml

  - job_name: "capitulo12"
    scrape_interval: 15s

    static_configs:
     - targets: ['localhost:8080']
    
```

La figura a continuación muestra el archivo en su totalidad. Recuerde mantener los espacios correspondientes para cada sección.

![](figura_12_02.png)

El código completo se muestra en la sección siguiente:


```yml

# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  
  - job_name: "capitulo12"
    scrape_interval: 15s

    static_configs:
     - targets: ['localhost:8080']
  
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090"]


```

Continuaremos trabajando con el proyecto del capítulo 11, donde se adjuntó el archivo de ejemplo **prometheus.yml** para su referencia.

Para crear el archivo UberJar y ejecutar el proyecto, utilice los siguientes comandos Maven en la consola:

```shell

mvn clean verify payara-micro:bundle

java -jar target/capitulo11-microbundle.jar --noHazelcast

```



La siguiente sección presenta todas las consultas realizadas en el capítulo 11 utilizando Curl. Es necesario que las ejecute para generar información correspondiente a las métricas

```shell

curl --location --request GET http://localhost:8080/api/estudiante

curl --location --request GET http://localhost:8080/api/faulttolerance

curl --location --request GET http://localhost:8080/health

curl --location --request GET http://localhost:8080/health/ready

curl --location --request GET http://localhost:8080/health/live

curl --location --request GET http://localhost:8080/health/startup

curl --location --request GET http://localhost:8080/api/estudiante

curl --location --request GET http://localhost:8080/metrics/

curl --location --request GET http://localhost:8080/metrics?scope=vendor

curl --location --request GET http://localhost:8080/metrics?scope=application

curl --location --request GET http://localhost:8080/metrics?scope=application&name=estudiantesFindAll

curl --location --request GET http://localhost:8080/api/estudiante/findbynombre?nombre=Ana

curl --location --request GET http://localhost:8080/api/estudiante/findbynombre?nombre=Maria

curl --location --request GET http://localhost:8080/api/estudiante/findbynombre?nombre=Luisa

curl --location --request GET http://localhost:8080/metrics?scope=application

curl --location --request GET http://localhost:8080/api/estudiante/1-2-3

curl --location --request GET http://localhost:8080/api/estudiante/7-8-5

curl --location --request GET http://localhost:8080/metrics?scope=application

curl --location --request GET http://localhost:8080/metrics?scope=application&name=estudianteCountFindByIdestudiante

curl --location --request GET http://localhost:8080/api/estudiante/histogram

```


Ahora, inicie Prometheus ejecutando el siguiente comando:


```shell

./prometheus

```


Abra su navegador e ingrese la siguiente dirección:


```
http://localhost:9090/targets?


```

En la figura siguiente , se puede observar el 'job' capitulo12 que se creó en los pasos anteriores


![](figura_12_03.png)



Acceda a http://localhost:8080/metrics  para visualizar los resultados de las métricas ejecutadas 

![](figura_12_04.png)


en el menú superior, seleccione opción 'Graph'

![](figura_12_05.png)


posteriormente, haga clic en 'Open Metrics Explorer'. Como se puede observar en la figura siguiente


![](figura_12_06.png)


Se presentarán las métricas de Microprofile Metrics, junto con otras métricas adicionales disponibles para su uso


![](figura_12_07.png)


Seleccione **estudianteCountFindByIdestudiante** y presione el botón 'Execute'. El resultado se mostrará en la parte inferior de la pantalla


![](figura_12_08.png)

Haga clic en la pestaña 'Graph' para visualizar un gráfico de los resultados 

![](figura_12_09.png)

Realice múltiples consultas, como las siguientes


```shell



curl --location --request GET http://localhost:8080/api/estudiante/1-2-3

curl --location --request GET http://localhost:8080/api/estudiante/7-8-5

curl --location --request GET http://localhost:8080/api/estudiante/6-8-2

```




Notará cambios en la gráfica basados en los resultados obtenidos

![](figura_12_10.png)

Establezca 5 minutos en la sección de tiempo, introduzca 15s en la casilla 'Res(s)' y realice varias consultas de nuevo

```shell

curl --location --request GET http://localhost:8080/api/estudiante/1-2-3

curl --location --request GET http://localhost:8080/api/estudiante/7-8-5

curl --location --request GET http://localhost:8080/api/estudiante/6-8-2

```

Observe qcómo se actualiza la gráfica tras las consultas


![](figura_12_11.png)




Prometheus permite personalización, permite agregar nuevos paneles para monitorear diversas métricas, crear reglas, además, cuenta con la opción de ejecutarse desde Docker, entre otras características.




## Grafana

Grafana es una herramienta de visualización que permite analizar e integrar datos con otras herramientas. Para instalar Grafana en Ubuntu siga los pasos a continuación. Si está utilizando otro sistema operativo, consulte la guía de instalación correspondiente.


```shell

sudo snap install grafana

```

Verifique que la instalación fue exitosa accediendo a la siguiente dirección en su navegador web [http://localhost:3000](http://localhost:3000).


Como se puede observar en la siguiente figura, se le solicitará que ingrese sus credenciales:

![](figura_12_12.png)


Por favor, proceda a ingresar las siguientes credenciales:


```

user: admin

password: admin


```


En la siguiente ventana, se le solicitará que cambie su contraseña. Introduzca una contraseña segura y presione el botón 'Save' 

![](figura_12_13.png)

Una vez hecho esto, será redirigido al portal principal. Haga clic en 'Add data Source '


![](figura_12_14.png)



De la lista de opciones disponibles, seleccione Prometheus  


![](figura_12_15.png) 


Especifique la URL de Prometheus [http://localhost:9090](http://localhost:9090) 


![](figura_12_16.png) 


Desplácese a la parte inferior y haga clic en el botón 'Save & Test'. Recibirá una notificación confirmando que los cambios se guardaron correctamente.




Haga clic en el botón '+' del menú izquierdo y seleccione la opción Dashboard 


![](figura_12_17.png) 


A continuación, seleccione 'Choose Visualization'

![](figura_12_18.png)  

Se mostrarán los diversos paneles de visualización 

![](figura_12_19.png)  


Seleccione la primera opción 'Query'. Observará que en el formulario que se muestra, 'Metrics' aparece como la primera opción

![](figura_12_20.png)


Las  métricas que desarrollamos en el capítulo 11 se presentan a continuación

![](figura_12_21.png)

Selecciona la métrica estudianteCountFindByIdestudiante que usamos con prometheus en los pasos anteriores. Observará que se genera una gráfica similar a la que obtuvimos anteriormente

![](figura_12_22.png)

Seleccione la opción visualización del menú izquierdo

![](figura_12_23.png)


Elija el tipo de gráfica que considere más adecuada de la lista disponible

![](figura_12_24.png)


En el menú izquierdo, seleccione 'General' y podrá agregar un título y una descripción 

![](figura_12_25.png)



Posteriormente configure la alarma. Para ello, haga clic en la opción 'Alarm' del menú izquierdo y posteriormente seleccione 'Create Alarm'

![](figura_12_26.png)




Seleccione el nombre de la alarma, determine la frecuencia con la que desea que se ejecute la duracción de cada ejecución. De manera predeterminada, se establece cada ejecutarse cada minuto.

![](figura_12_27.png)


Haga clic en el botón 'Save' ubicado en la parte superior

![](figura_12_28.png)


Ingrese 'Capitulo 12 Dashboard' como el nombre del 'dashboard' y presione el botón 'Save'

![](figura_12_29.png)


Puede modificar el intervalo de ejecución haciendo clic y seleccionándolo de la lista  

![](figura_12_30.png) 

Al realizar múltiples consultas el Endpoint, puede observar las modificaciones en la gráfica

```shell

curl --location --request GET http://localhost:8080/api/estudiante/1-2-3

curl --location --request GET http://localhost:8080/api/estudiante/7-8-5

curl --location --request GET http://localhost:8080/api/estudiante/6-8-2

```


### Administrar Grafana


En el menú izquierdo, seleccione 'Configuration' y posteriormente 'Datasources', utilizando la URL [http://localhost:3000/datasources](http://localhost:3000/datasources) 


![](figura_12_31.png) 


Se mostrará 'datasoruces prometheus'. Si da clic en él, podrá modificarlo.


![](figura_12_32.png)

En el menú izquierdo, seleccione 'Alert Rules' para visualizar las reglas que has creado.

![](figura_12_33.png)


## Resumen

En este capítulo, hemos explorado el uso de Prometheus y Grafana para supervisar microservicios. En el próximo capítulo se presentará ejemplos prácticos de cómo utilizar JMeter.

