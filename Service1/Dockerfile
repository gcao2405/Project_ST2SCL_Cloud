# FROM openjdk:8-jdk-alpine
FROM khipu/openjdk17-alpine
VOLUME /tmp
EXPOSE 8080
ADD ./build/libs/Service1-0.0.1-SNAPSHOT.jar app.jar
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]