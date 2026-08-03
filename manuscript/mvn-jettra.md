# mvn-jettra

# Creador de Plugins Jettra (PluginCLI)
Guía completa de generación, instalación y administración autónoma de módulos JettraFlux
Este documento detalla el uso del PluginCLI, un entorno aislado para la generación 100% autónoma de proyectos Jettra mediante comandos de consola, facilitando la creación, instalación y eliminación de plugins JettraFlux.

## Introducción y Conceptos Clave
El sistema de plugins permite reutilizar funcionalidades sin reescribir componentes, ahorrando tiempo de desarrollo y pruebas. A partir de un proyecto Java Maven, se genera una estructura simplificada (.jar) que contiene clases y archivos de propiedades necesarios para integrarse como una dependencia en proyectos superiores.
JettraHub
Es el directorio autorizado para plugins de Jettra donde se pueden compartir o descargar funcionalidades adaptables. Su URL oficial es [https://github.com/jettraframework/jettrahub](https://github.com/jettraframework/jettrahub)


## Herramienta CLI: mvn-jettra
La administración se realiza mediante el script mvn-jettra.sh, el archivo de script utiliza JettraAppServer para interactuar con el sistema.  

Estructura del archivo (mvn-jettra.sh)

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


# **. Generación de Plugins (`generate-plugin`)**

Permite convertir un proyecto Jettra en un plugin independiente.

## **Parámetros Principales**

* **\-path**: Directorio donde se generará el plugin.  
* **\-name**: Nombre del plugin.  
* **exclude-package**: Paquetes a omitir (ej. `com.avbravo.general`).  
* **exclude-class**: Clases específicas a excluir.  
* **incluye-test**: Define si se integran las pruebas (`yes`|`no`).

## **Ejemplo de Sintaxis**

`./mvn-jettra generate-plugin -path ~/Descargas -name MiPlugin exclude-package com.avbravo.general incluye-test yes`

## **¿Qué ocurre internamente?**

1. **Propiedades**: Se generan archivos `messages-<nombre-plugin>.properties` y se actualizan las anotaciones `@InjectProperties`.  
2. **Rutas**: Se modifica el path de los componentes `@Page` (ej. `/person` pasa a `/miplugin/person`) para evitar conflictos.  
3. **Roles**: Se cambian las restricciones de `jcf.AppRole` a `pjc.<nombre-plugin>.AppRole`.  
4. **Descriptor**: Se crea el archivo `plugin-descriptor.md` en `/resources` con información de menús y restricciones.

# **4\. Instalación de Plugins (`install-plugin`)**

Este comando integra el plugin en el proyecto principal.

* **Mecánica**: Compila el plugin, lo añade como dependencia al `pom.xml` e inyecta los menús en `TemplatePage.java` sin duplicar código.  
* **Configuración de Roles**: Genera el archivo `plugin-config.json`. Este archivo permite mapear los roles del plugin con los de la aplicación actual (sinónimos).

**Ejemplo de uso:**  
`./mvn-jettra install-plugin MiPlugin`

# **5\. Configuración de Maven (`settings.xml`)**

Para un funcionamiento óptimo, se debe configurar el archivo `settings.xml` de Maven definiendo el repositorio local y activando el perfil de Jettra con la versión de Java correspondiente (Java 25).l

jettra-profile  

