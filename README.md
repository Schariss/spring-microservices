# Spring Microservices in Action

# Content

1. A Spring Cloud Config server that is deployed as Docker container and can manage a services configuration 
information using a file system/ classpath or GitHub-based repository.
2. A licensing service that will manage licensing data.
3. A Postgres SQL database used to hold the data.

## Initial Configuration
1. Apache Maven (http://maven.apache.org)
2. jdk 11
3. Docker

## How To Use

To clone and run this application, you'll need [Git](https://git-scm.com), [Maven](https://maven.apache.org/), 
[Java 11](https://www.oracle.com/technetwork/java/javase/downloads/jdk11-downloads-5066655.html). 

From your command line:

```bash
# To install images locally execute the following command:
$ mvn clean install --skip-tests

# Run the following command: 
$ docker-compose -f docker/docker-compose.yml up
```

## Refresh scope
When custom configuration parameters change in your Git repository, the Spring Cloud Config Server automatically detects 
the updates (but not if you use File system config - native - backend). For example, if you query http://localhost:8071/licensing-service/dev, 
you’ll see the latest modifications. 

However, the licensing-service client loads its configuration only at startup, so it won’t automatically reflect these 
changes. To update the client with the latest configuration, you need to call the /actuator/refresh endpoint.

Keep in mind that if you have multiple instances of the service running, you must call /refresh on each instance. 
One way to achieve this is by using a service discovery mechanism to retrieve all instances and send a POST request 
to /refresh for each one. Another option is to use Spring Cloud Bus, which can broadcast configuration updates across all 
instances, but this requires an additional middleware, such as RabbitMQ.