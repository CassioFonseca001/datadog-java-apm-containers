# OpenJ9 JVM Datadog APM Containers
Repo for testing/helping Datadog customer's get started with OpenJ9 Java APM using Containers.

***Note: THIS IS NOT MEANT TO BE A DISTRIBUTED TRACING EXAMPLE, SIMPLY HOW TO GET STARTED WITH DATADOG APM WITH THE OpenJ9 JVM AND DOCKER***

# Objectives
- Get a sample Java application, running in a container using [IBM & Eclipse's JVM Distribution (OpenJ9)](https://en.wikipedia.org/wiki/OpenJ9), running and reporting Java APM metrics to Datadog via the Docker Datadog Agent.
- Follow and adapt the [Datadog Java APM instructions](https://docs.datadoghq.com/tracing/languages/java/)
for container usage.

# Noteworthy
- This small project is for demonstration purposes only.
- It does not make use any container orchestrator.
- It's original intent was to be used on a developer's local machine.

# Project Structure
- [agent](./agent): Contains a [dockerfile](./agent/Dockerfile) and [datadog.conf](./agent/datadog.conf) for building the containerized Datadog agent locally w/ some minimal configuration.
- [datadog](./datadog): Contains the `dd-java-agent.jar` version `0.3.0` as of 2018/02/12. Ideally this gets pulled in at build time and incorporated into the gradle build process.
- [gradle](./gradle): for gradle wrapper (run across multiple platforms)
- [src](./src/main/java/hello): Contains our very simple Sprint Boot application code.
- [.gitignore](./.gitignore): For ignoring files and directories generated by build tools
- [build.gradle](./build.gralde): Gradle build definition
- [Dockerfile](./Dockerfile): Primary Dockerfile for our simple Java app
  - Uses `ibmjava:jre` - e.g. OpenJ9 JVM. Copies over our WAR, exposes 8080 and sets several necessary Datadog Java APM properties
- [Dockerfile-openjdk](./Dockerfile-openjdk): A test bed for working with OpenJDK versus OpenJ9.
- [gradlew](./gradlew) and [gradlew.bat](./gradlew.bat): Gradle wrapper files - used for building Java project.

# Prerequisites
- Install Java (Either OpenJ9 or your favorite flavor (OpenJDK, Oracle, etc)) for your platform
- Install Docker for your platform

# Send APM Metrics to Datadog (OpenJ9)
## Run the Datadog Dockerized Agent
- Run to build the image: `docker build -t dd-agent ./agent/`
- Run the following, replacing `{your_api_key_here}` with your own DD API key.
```
docker run -d --rm --name dd-agent \
  -v /var/run/docker.sock:/var/run/docker.sock:ro \
  -v /proc/:/host/proc/:ro \
  -v /sys/fs/cgroup/:/host/sys/fs/cgroup:ro \
  -e API_KEY={your_api_key_here} \
  -e DD_APM_ENABLED=true \
  -p 8126:8126/tcp \
  -e SD_BACKEND=docker \
  -e LOG_LEVEL=DEBUG \
  -e DD_LOGS_STDOUT=yes \
  -e DD_PROCESS_AGENT_ENABLED=true \
  dd-agent
```

## Run the Java example
- Run `./gradlew build` (or `gradlew.bat` if on windows)
- Run to build the docker image: `docker build -t dd-java-apm-hello-world .`
- Run to start the container:
```
docker run -d -p 8080:8080 --rm --name dd-java-apm dd-java-apm-hello-world \
-e TAGS=host:dd-java-apm-demo,env:demo
```
- Run to see the containers running: `docker ps`
- Run to see container logs: `docker logs dd-java-apm`
- Run to get to bash prompt for the container: `docker exec -it dd-java-apm /bin/bash`
- Run to stop the container: `docker stop dd-java-apm`
- Run to remove the container: `docker rm dd-java-apm`

## See APM in Datadog
Visit [Datadog APM env:demo](https://app.datadoghq.com/apm/services?env=demo) and the `dd-java-apm-example-openj9` service should be listed.

# Send APM Metrics to Datadog (OpenJDK)
- Run the [Datadog Dockerized Agent as described in the OpenJ9 Section](#run-the-datadog-dockerized-agent)
- Uncomment lines 28 through 41 in [Application.java](./src/main/java/hello/Application.java#L28-L41) to enable additional URLs to gather metrics and traces from
- Run `./gradlew build` (or `gradlew.bat` if on windows)
- Run to build the docker image: `docker build -t dd-java-apm-hello-world-openjdk -f Dockerfile-openjdk .`
- Run to start the container:
```
docker run -d -p 8081:8080 --rm --name dd-java-apm-openjdk dd-java-apm-hello-world-openjdk \
-e TAGS=host:dd-java-apm-demo-openjdk,env:demo
```

# Notes on Java APM Support
As of 2018/02/12 Java APM has out of the box support for many popular Java frameworks, app servers, and data stores - check the [Datadog APM docs for the up to date list](https://docs.datadoghq.com/tracing/languages/java/#integrations).
- Java Servlet Compatible - Many application servers are Servlet compatible, such as Tomcat, Jetty, Websphere, Weblogic, etc. Also, frameworks like Spring Boot and Dropwizard inherently work because they use a Servlet compatible embedded application server.
- OkHTTP | 3.x
- Apache HTTP Client | 4.3 +
- JMS 2 | 2.x
- JDBC | 4.x
- MongoDB | 3.x
- Cassandra | 3.2.x

# Resources
- https://github.com/DataDog/docker-dd-agent
- https://docs.datadoghq.com/tracing/languages/java/#setup
- https://docs.datadoghq.com/api/?lang=bash#tracing
