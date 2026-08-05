
# Modificar el archivo mvn-flux.md 

y la sección que genera los archivos mvn-flux y mvn-jettra reemplazarlos por 

```shell

mvn exec:java -Dexec.mainClass="io.jettra.server.JettraServer" -Dexec.args="-generate-flux-jettra-sh"

```
Esta instrucción genera de manera automática los archivos mvn-flux y mvn-jettra.
