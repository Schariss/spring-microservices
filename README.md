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

## HashiCorp Vault
Vault is another backend repository that we can use as a spring cloud config server backend. It's a tool allows us 
to securely access secrets. We can define secrets as any piece of info we want to restrict or control access to, such as 
passwords, certificates, API keys, and so forth.
1. Launch docker-compose-infra.yml with :
```bash
docker-compose -f docker/docker-compose-infra.yml up
```
This command will launch a postgres DB with an instance of a Vault dev server.

Go to Secretes Engine and create a new engine called _licensing-service_, choose kv v2, then create two new secrets with
the names _licensing-service_ and _licensing-service-dev_ that correspond respect. to default and dev profiles, with
the following params: vault.name and vault.pwd.

The configuration is available under:
```bash
curl -X "GET" http://127.0.0.1:8200/v1/licensing-service/data/licensing-service -H "X-Vault-Token: myroot"
```
```bash
curl -X "GET" http://127.0.0.1:8200/v1/licensing-service/data/licensing-service-dev -H "X-Vault-Token: myroot"
```

__Response example__:
```json
{
    "request_id": "22483309-1e20-5163-017c-951f88a5b3cc",
    "lease_id": "",
    "renewable": false,
    "lease_duration": 0,
    "data": {
        "data": {
            "vault.name": "dev",
            "vault.pwd": "MyLocalPWD"
        },
        "metadata": {
            "created_time": "2024-09-29T08:20:01.699214334Z",
            "custom_metadata": null,
            "deletion_time": "",
            "destroyed": false,
            "version": 1
        }
    },
    "wrap_info": null,
    "warnings": null,
    "auth": null,
    "mount_type": "kv"
}
```
From the config server, the properties are available under:
- http://localhost:8071/licensing-service/default
- http://localhost:8071/licensing-service/dev

__Response example__:
```json
{
    "name": "licensing-service",
    "profiles": [
        "dev"
    ],
    "label": null,
    "version": null,
    "state": null,
    "propertySources": [
        {
            "name": "vault:licensing-service-dev",
            "source": {
                "vault.name": "dev",
                "vault.pwd": "MyLocalPWD"
            }
        },
        {
            "name": "vault:licensing-service",
            "source": {
                "vault.name": "default",
                "vault.pwd": "pwd"
            }
        },
        {
            "name": "https://github.com/username/spring-ms-config.git/licensing-service-dev.properties",
            "source": {
                "example.property": "I AM DEV",
                "spring.datasource.url": "jdbc:postgresql://${DB_HOST_NAME:localhost}:5432/ostock_dev",
                "spring.datasource.username": "postgres",
                "spring.datasource.password": "postgres"
            }
        },
        {
            "name": "https://github.com/username/spring-ms-config.git/licensing-service.properties",
            "source": {
                "example.property": "I AM THE DEFAULT VALUE",
                "spring.jpa.hibernate.ddl-auto": "none",
                "spring.jpa.database": "POSTGRESQL",
                "spring.datasource.platform": "postgres",
                "spring.jpa.show-sql": "true",
                "spring.jpa.hibernate.naming-strategy": "org.hibernate.cfg.ImprovedNamingStrategy",
                "spring.jpa.properties.hibernate.dialect": "org.hibernate.dialect.PostgreSQLDialect",
                "spring.database.driverClassName": "org.postgresql.Driver",
                "spring.datasource.testWhileIdle": "true",
                "spring.datasource.validationQuery": "SELECT 1",
                "management.endpoints.web.exposure.include": "*",
                "management.endpoints.enabled-by-default": "true",
                "local.lang": "en"
            }
        }
    ]
}
``` 
