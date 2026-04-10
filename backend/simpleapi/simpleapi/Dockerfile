# Build stage

# JDK 21 avec nom du stage
FROM eclipse-temurin:21-jdk-alpine AS myapp-build
ENV MYAPP_HOME=/opt/myapp

# Working directory
WORKDIR $MYAPP_HOME

# install maven
RUN apk add --no-cache maven

# Copy pom.xml and source code inside docker image, then build the application
COPY pom.xml .
COPY src ./src

# Télécharger les dépendances et construire l'application sans exécuter les tests
RUN mvn package -DskipTests

# Run stage

# Image qui ne contient que JRE (Java Runtine Environment)
#ok pour lancer Java mais pas pour compiler
FROM eclipse-temurin:21-jre-alpine
ENV MYAPP_HOME=/opt/myapp

# Working directory
WORKDIR $MYAPP_HOME
# Copier le jar construit dans l'étape précédente
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar

# Exécuter l'application au lancement du conteneur
ENTRYPOINT ["java", "-jar", "myapp.jar"]
