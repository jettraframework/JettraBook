# mvn-jettra

# Creador de Plugins Jettra (PluginCLI)
Guía completa de generación, instalación y administración autónoma de módulos JettraFlux
Este documento detalla el uso del PluginCLI, un entorno aislado para la generación 100% autónoma de proyectos Jettra mediante comandos de consola, facilitando la creación, instalación y eliminación de plugins JettraFlux.


## Descripcion
* Facilita la creación, instalación y eliminación de plugins JettraFlux a través de la terminal mediante una generación 100% autónoma y plantillas nativas, garantizando un entorno aislado.

* A partir de un proyecto Java Maven se genera una estructura simplificada llamada plugin que se almacena en un archivo .jar

* Este archivo jar contiene las clases necesarias  y archivos properties para integrar la funcionalidad en un proyecto superior que lo usara como una simple dependencia.

* Permite la reutilización de funcionalidades sin necesidad de reescribir los componentes y su comportamiento.

* Un plugin  ahorra tiempo de desarrollo y pruebas.

* Tus proyectos personales/empresariales los puedes convertir en plugin

* Para uso interno o compartirlos con la comunidad mediante jettrahub.

# JettraHub
Es el directorio autorizado para plugins de Jettra. https://github.com/jettraframework/jettrahub
 que se pueden reutilizar y adaptar a sus necesidades particulares.



## Introducción y Conceptos Clave
El sistema de plugins permite reutilizar funcionalidades sin reescribir componentes, ahorrando tiempo de desarrollo y pruebas. A partir de un proyecto Java Maven, se genera una estructura simplificada (.jar) que contiene clases y archivos de propiedades necesarios para integrarse como una dependencia en proyectos superiores.
JettraHub
Es el directorio autorizado para plugins de Jettra donde se pueden compartir o descargar funcionalidades adaptables. Su URL oficial es [https://github.com/jettraframework/jettrahub](https://github.com/jettraframework/jettrahub)


## Herramienta CLI: mvn-jettra
La administración se realiza mediante el script mvn-jettra.sh, el archivo de script utiliza JettraAppServer para interactuar con el sistema.  

Contenido del archivo (mvn-jettra)

```shell

#!/bin/bash

if [ "$1" = "-jettra" ]; then

    shift
fi

mvn -q exec:java -Dexec.mainClass="io.jettra.server.cli.PluginCLI" -Dexec.args="$*"

```

## **Comandos Disponibles**

| Comando | Descripción |
| :---- | :---- |
| `help` | Muestra la ayuda de los comandos disponibles. |
| `list-plugin` | Lista los plugins registrados en JettraHub. |
| `get-plugin` | Descarga y configura un plugin desde el repositorio en el `pom.xml`. |
| `generate-plugin` | Genera un nuevo plugin a partir de un proyecto existente. |
| `install-plugin` | Configura el menú y roles del plugin en el proyecto destino. |
| `remove-plugin` | Elimina las configuraciones y dependencias del plugin en el proyecto. |


---
# Help
Muestra en consola la ayuda de los comandos disponibles

```shell
./mvn-jettra help
```

---

# list-plugin
Muestra los repositorios alojados en  [https://github.com/avbravo/jettrahub.git](https://github.com/avbravo/jettrahub.git)

```shell
./mvn-jettra list-plugin
```

---
# get-plugin
Obtener plugins del repositorio
Realiza los siguientes procesos
1. Verifica en github el archivo de configuración <nombre-plugin>.json en Jettrahub y configura el archivo pom.xml con  el plugin

Para obtener el plugin JettraFluxExample del repositorio central ejecute

```shell
./mvn-jettra get-plugin JettraFluxExample
```
2. De manera automática añade el repositorio jitpack.io

```xml
<repositories>
   <repository>
       <id>jitpack.io</id>
       <url>https://jitpack.io</url>
   </repository>
</repositories>
```
3. Proceda a instalar el plugin
   
```shell
./mvn-jettra installJettraFluxExample
```

---
# **. Generación de Plugins (`generate-plugin`)**

Permite convertir un proyecto Jettra en un plugin independiente.

## **Parámetros Principales**

* **\-path**: Directorio donde se generará el plugin.  
* **\-name**: Nombre del plugin.  
* **exclude-package**: Paquetes a omitir (ej. `com.avbravo.general`).  
* **exclude-class**: Clases específicas a excluir.  
* **incluye-test**: Define si se integran las pruebas (`yes`|`no`).

## generate-plugin -path -name
### **Ejemplo de Sintaxis**

**Sintaxis Recomendada**
Utiliza parámetros explícitos para mayor seguridad y control en la ubicación del proyecto:

```shell
./mvn-jettra generate-plugin -path -name [opciones]
```

Ejemplo
```shell
./mvn-jettra generate-plugin -path ~/Descargas -name MiPlugin 
```

## generate-plugin exclude-plugin exclude-package

### exclude-plugin
Utiliza parámetros exclude-plugin para excluir plugins añadidos que no seran pasados al nuevo plugin.
```shell
./mvn-jettra generate-plugin -path /home/myuser/Descargas -name MiPlugin exclude-plugin plugin1,plugin2 
```
### exclude-package
Construye el plugin directamente en el directorio de trabajo actual si omitas -path:

```shell
./mvn-jettra generate-plugin -path /home/myuser/Descargas -name MiPlugin exclude-package com.avbravo.general, com.avbravo.prueba  incluye-test yes
```

### exclude-class

Permite indicar las clases que serán excluidas
```shell
./mvn-jettra generate-plugin -path /home/myuser/Descargas -name MiPlugin exclude-class Clase1.java, Clase2.java
```

### incluye-test
Incluye los test del plugin : yes | no
```shell
./mvn-jettra generate-plugin -path /home/myuser/Descargas -name MiPlugin    includes-test  yes
```

----
## plugin-descriptor.md
Archivo con información del plugin.

## **¿Qué ocurre internamente?**

1. **Propiedades**: Se generan archivos `messages-<nombre-plugin>.properties` y se actualizan las anotaciones `@InjectProperties a @@InjectProperties(name = "messages-<nombre-plugin>")`.
2. Se genera en el plugin el archivo **plugin-descriptor.md**
