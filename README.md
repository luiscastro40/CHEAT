#  Docker

## Pré requisitos

- Maven 3

- Java 8

- Docker 1.13.0+



## Preparar o ambiente

Make sure you have `Maven` installed. Execute the following maven command from the directory of the 
parent project, `docker-example`:

```
mvn clean package dockerfile:build
```
It will create the Spring Boot executable JAR,`docker-example-1.0.jar`, under `docker-example/target` 
folder.

No caso de ser preciso desligar o servidor
```
Ctrl+C
```

## Docker (linha de comandos)
- Container do Postgres
```
docker run -it -d \
    
    --name docker-postgres \
    
    -e POSTGRES_DB=db \
   
    -e POSTGRES_USER=postgres \
   
    -e POSTGRES_PASSWORD=postgres
  
    postgres:10.4

```

- Container do Docker image, `docker-example`, by executing the 
[`docker run`](https://docs.docker.com/engine/reference/run/) command from the terminal:
```
docker run --rm --name=cheetos docker-example
```
- Ligação entre os containers

```
docker run -it
     
   --link docker-postgres
     
   -p 8080:8080
    
   emmanuelneri/spring-boot-docker-app

```
To display the built docker images
```
docker images ls
```
```
CONTAINER ID        IMAGE                                 COMMAND                  CREATED             STATUS              PORTS                    NAMES

3b7f0cfeceaf        emmanuelneri/spring-boot-docker-app   "java -Djava.securit…"   8 minutes ago       Up 7 seconds        0.0.0.0:8080->8080/tcp   springbootdocker_docker-app_1

7f01ce21cb11        postgres:10.4                         "docker-entrypoint.s…"   8 minutes ago       Up 7 seconds        5432/tcp                 springbootdocker_docker-postgres_1

```

Options
* `--rm` option automatically clean up the container and remove the file system when the container exit.
* `--name` option names the Docker container as `cheetos`. In absence of the `--name` option, the Docker generates a 
random name for your container.
* [`-p 8080:8080`](https://docs.docker.com/engine/reference/run/#expose-incoming-ports) option publishes all 
exposed ports to the host interfaces. In our example, it is port `8080` is both `hostPort` and `containerPort` 
* -it: Executa o container com o shell iterativo com o container (it = interactive ) 
* –link: Criar link com outros containers, no caso com o container da base de dados;
* -p: Configura a porta a ser executado o serviço;
* -d: Mantém a execução do container mesmo que fechado a janela (d = ativando Detached mode);
* -e: Define configuração e ambiente (e = environment).

## Docker (IDE- Compose)
Ferramenta do próprio Docker para execução de múltiplos containers. Assim, basta criar o arquivo docker-compose.yml com as configurações abaixo e executar o comando docker-compose up.
```
docker-compose up
```

## Docker Commands
### List Container
Run the [`docker ps`](https://docs.docker.com/v1.11/engine/reference/commandline/ps/) to list all the containers.
To see all running containers, execute the following command:
```
bash-3.2$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
d03854fb7779        docker-example      "java -Djava.security"   7 seconds ago       Up 6 seconds        0.0.0.0:8080->8080/tcp   cheetos
bash-3.2$ 
```
To see all running containers including the non-running ones, execute the following command:
```
bash-3.2$ docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS                         PORTS                    NAMES
d03854fb7779        docker-example      "java -Djava.security"   About a minute ago   Up About a minute              0.0.0.0:8080->8080/tcp   cheetos
28b2cff9e7e6        docker-example      "java -Djava.security"   About an hour ago    Exited (0) About an hour ago                            indra1
d2720676c932        nginx               "nginx -g 'daemon off"   4 months ago         Exited (0) 4 months ago                                 webserver
```

### Remove Container
To remove a Docker container, execute [`docker rm`](https://docs.docker.com/v1.11/engine/reference/commandline/rm/) 
command. This will remove a non-running container.
```
bash-3.2$ docker rm indra1
indra1 
```
To forcefully remove a running container
```
bash-3.2$ docker rm -f cheetos
cheetos
```

### Stop Container
To stop a container, execute [`docker stop`](https://docs.docker.com/v1.11/engine/reference/commandline/stop/)
```
command:
bash-3.2$ docker stop cheetos
cheetos
```

## Docker Maven Plugin Setup
We are using [`Spotify's Docker Maven Plugin`](https://github.com/spotify/docker-maven-plugin). It's relatively easy to
setup in your Maven `pom.xml`. Complexity rises when you specify the plugin in the parent POM. Here are the changes to
the parent `pom.xml`
```
    <build>
        <plugins>
            <plugin>
                <groupId>com.spotify</groupId>
                <artifactId>docker-maven-plugin</artifactId>
                <version>0.4.13</version>
                <configuration>
                    <skipDockerBuild>true</skipDockerBuild>
                </configuration>
            </plugin>
        </plugins>
    </build>
```
The `skipDockerBuild` tag is set to `true` in order to skip the docker build in the parent pom.

Changes to the child `pom.xml` where Spring Boot JAR gets created:
```
    <build>
        <plugins>
            <plugin>
                <groupId>com.spotify</groupId>
                <artifactId>docker-maven-plugin</artifactId>
                <configuration>
                    <skipDockerBuild>false</skipDockerBuild>
                    <imageName>${docker.image.name}</imageName>
                    <dockerDirectory>${basedir}/src/main/docker</dockerDirectory>
                    <resources>
                        <resource>
                            <targetPath>/</targetPath>
                            <directory>${project.build.directory}</directory>
                            <include>${project.build.finalName}.jar</include>
                        </resource>
                    </resources>
                </configuration>
            </plugin>
        </plugins>
    </build>
```
Here is the `skipDockerBuild` tag is set to `false` to override the parent flag.

##### Configuration Tags
* `imageName` specifies the name of our example Docker image, e.g, `docker-example`
* `dockerDirectory` specifies the location of the `Dockerfile`. The contents of the dockerDirectory will be 
copied into `${project.build.directory}/docker`. A [`Dockerfile`](https://docs.docker.com/engine/reference/builder/)
specfies all the instructions to be read by Docker while building the image.
* `include` specfies the resources to be included, which ion our case is `docker-example-service-1.0.jar`

## DockerFile
Here is the content of the example `Dockerfile`.

```
FROM openjdk:10-jdk
VOLUME /tmp
COPY target/induction-0.0.1-SNAPSHOT.jar app.jar
ENTRYPOINT ["java", "-Djava.security.egd=file:/dev/./urandom", "-Dspring.profiles.active=docker", "-jar", "/app.jar"]
```

```
FROM frolvlad/alpine-oraclejdk8:slim
VOLUME /tmp
ADD docker-example-service-1.0.jar app.jar
RUN sh -c 'touch /app.jar'
EXPOSE 8080
ENV JAVA_OPTS=""
ENTRYPOINT ["java", "-Djava.security.egd=file:/dev/./urandom", "-Dapp.port=${app.port}", "-jar","/app.jar"]
LABEL maintainer "Indra Basak"
```

#### Instruction
* `FROM` instruction sets the Base Image for subsequent instructions. FROM must be the first non-comment 
instruction in the Dockerfile.
* `VOLUME` instruction creates a mount point with the specified name.
* `ADD` instruction copies from `<src>` and adds them to the filesystem of the image at the path `<dest>`.
* `COPY` instruction copies from `<target>` and adds them to the filesystem of the image at the path `<dest>`.
* `RUN` instruction executes the command on top of the current image.
* `EXPOSE` instruction informs Docker that the container listens on the specified network ports at runtime.
* `ENV` instruction sets the environment variable
* `ENTRYPOINT` allows you to configure a container that will run as an executable.
* `LABEL` instruction adds metadata to an image.

You can find out more about Docker instructions [`here`](https://docs.docker.com/engine/reference/builder/#usage)

[travis-badge]: https://travis-ci.org/indrabasak/docker-example.svg?branch=master
[travis-badge-url]: https://travis-ci.org/indrabasak/docker-example/

```
