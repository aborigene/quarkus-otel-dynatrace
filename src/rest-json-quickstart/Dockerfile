#building java
FROM adoptopenjdk/maven-openjdk11 as BUILD
COPY src /usr/src/app/src
COPY ./pom.xml /usr/src/app
WORKDIR /usr/src/app
RUN mvn package -DskipTests
RUN mv target target_java

#building native
FROM quay.io/quarkus/ubi-quarkus-native-image:22.0-java11 AS BUILD_NATIVE
#ENV MAVEN_OPTS="-Xmx6000m"
#ENV JAVA_OPTS="-Xmx6000m"
COPY --chown=quarkus:quarkus mvnw /code/mvnw
COPY --chown=quarkus:quarkus .mvn /code/.mvn
COPY --chown=quarkus:quarkus pom.xml /code/
USER quarkus
WORKDIR /code
RUN chmod +x mvnw
RUN ls -l
RUN ./mvnw -B org.apache.maven.plugins:maven-dependency-plugin:3.1.2:go-offline
COPY src /code/src
RUN ./mvnw package -X -Pnative -DskipTests

#FROM quay.io/quarkus/quarkus-micro-image:1.0
FROM openjdk:11.0.13-jdk
WORKDIR /work/
#copying java
COPY --from=BUILD --chown=1001 /usr/src/app/target_java/quarkus-app/ /work/
RUN mv quarkus-run.jar application.jar

#copying native
COPY --from=BUILD_NATIVE /code/target/*-runner /work/application
RUN mkdir /work/config

#copying startup.sh
COPY ./startup.sh /work/startup.sh

RUN chmod 775 /work
EXPOSE 8087
CMD ["bash", "startup.sh"]
