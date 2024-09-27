## Spring Microservices in Action

# Content

1.  A Spring Cloud Config server that is deployed as Docker container and can manage a services configuration information using a file system/ classpath or GitHub-based repository.
2.  A licensing service that will manage licensing data.
3.  A Postgres SQL database used to hold the data.

## Initial Configuration
1. Apache Maven (http://maven.apache.org)
2. jdk 11
3. Docker

## How To Use

To clone and run this application, you'll need [Git](https://git-scm.com), [Maven](https://maven.apache.org/), [Java 11](https://www.oracle.com/technetwork/java/javase/downloads/jdk11-downloads-5066655.html). From your command line:

```bash
# To install images locally execute the following command:
$ mvn clean install --skip-tests

# Run the following command: 
$ docker-compose -f docker/docker-compose.yml up
```
