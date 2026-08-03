
Capitulo 2 modificar el final name y modificar el Dockerfile

FROM payara/micro:6.2024.12-jdk21
COPY target/${project.artifactId}.war $DEPLOY_DIR
#COPY target/notaria-0.1-SNAPSHOT.war $DEPLOY_DIR


Cambiar el capitulo 2 con  el uso de PayaraStarter

* revisar si funciona el bundle

* :start
