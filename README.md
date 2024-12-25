# Ktor with Postgresql Raspberry Pi 5 docker setup
This repo will show you how to setup raspberry pi 5 server using ktor and postgresql.

## Versions
- Docker version `27.4.1`, build `b9d17ea`
## Setup
Firstly we are going to create a docker network for the postgres and ktor. Execute this command:
```bash
docker network create <network-name>
```
### Postgres part
```bash
docker run -d \
  --name postgres-db \
  --network motopark-network \
  -e POSTGRES_USER=admin \
  -e POSTGRES_PASSWORD=password \
  -e POSTGRES_DB=motopark \
  -p 5432:5432 \
  postgres:15-alpine
```
Now the posgres should be running. You can check it by using `docker ps -a`. Output should look like this:
```
CONTAINER ID   IMAGE                COMMAND                  CREATED          STATUS                      PORTS                                       NAMES
812322b762c6   postgres:15-alpine   "docker-entrypoint.s…"   1 minute ago      Up 1 minutes               0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   postgres-db
```
### Ktor part
1. Add this configuration to the `build.gradle.kts` for building the FAT JAR
```kotlin
tasks.create<Jar>("fatJar") {
    archiveClassifier.set("all")
    duplicatesStrategy = DuplicatesStrategy.EXCLUDE
    manifest {
	attributes(
	     "Main-Class" to "org.example.ApplicationKt"
	)
    }
    from(sourceSets.main.get().output)

    dependsOn(configurations.runtimeClasspath)
    from({
	configurations.runtimeClasspath.get().map { if (it.isDirectory) it else zipTree(it) }
    })
}
```
2. Before building the FAT JAR, you should change the connection string for the db. Here is example:
```kotlin
jdbc:postgresql://postgres-db:5432/<database-name>
```
3. Now build the project using `./gradlew build` and run `./gradlew fatJar` for creating the FAT JAR
4. We need to build the Docker image now from the FAT JAR. Start by creating the `Dockerfile` in the root of project and paste this content inside.
```dockerfile
FROM arm64v8/eclipse-temurin:17-jdk                //Java

WORKDIR /app

COPY build/libs/<example-project>-all.jar app.jar

EXPOSE 8080                                        //Ktor exposed port

CMD ["java","-jar","app.jar"]
```
5. Now we can create the Docker image.
```bash
docker build -t <docker-image-name> .
```
6. Let`s create the docker container now.
```bash
docker run -d -p 8080:8080 --name <docker-container-name> <docker-image-name>
```
7. And it's done. You can check if the docker is running by `docker ps -a`. Output should look like this.
```bash
CONTAINER ID   IMAGE                COMMAND                  CREATED          STATUS                         PORTS                                       NAMES
c15571c61edb   <docker-iamge-name>  "/__cacert_entrypoin…"   28 minutes ago   Up 28 minutes                  0.0.0.0:8080->8080/tcp, :::8080->8080/tcp   <docker-container-name>
812322b762c6   postgres:15-alpine   "docker-entrypoint.s…"   4 hours ago      Up 40 minutes                  0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   posgres-db
```
Thank you for your time. If there are some issues, you can create an issue request here on github, and i will try to help you ✌️
