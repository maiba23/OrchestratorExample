# Orchestrator

Sequences and Schedules processes, validates execution, records and reports progress.

## First-time Setup

To run this example, you need to add a `.env` file to the `OrchestratorExample`
folder which contains credentials. This file will be ignored by source control.
It should contain the following tokens:

```
CONFIGREPO_URI=https://gitlab.com/reductech/forensync/configuration.git
CONFIGREPO_USERNAME=GitLabTokenUser
CONFIGREPO_PASSWORD=GitLabToken/PAT
```

Docker needs to be installed locally. If using windows, switch to linux containers.

Log into the gitlab container registry:

```
docker login registry.gitlab.com -u <username> -p <PersonalAccessToken>
```

In powershell run:

```powershell
PS> cd .\OrchestratorExample\
PS> Expand-Archive .\data\mariadb.zip .\data\mariadb
PS> Expand-Archive .\data\vaultdata.zip .\data\vault
```

This will setup all the required data volumes for development.

## Running the Example

```powershell
PS> cd .\OrchestratorExample
PS> docker-compose up -d
```

## Unlocking Vault

The root user token is `s.cEiqBZRlRRm0HfChUWUBlmyr` and this is used in
the `Settings.yml` file in the Config server.

In order to login you need to unseal the vault.

- Go to the [Vault UI](http://localhost:8200/ui) and use the unseal key
  `O0wAtF/M6PRp3Djdl5lo5+7cERaUNQlJAwQM8dMiBsU=` to unseal it.
- You can now login and run queries
- Use the `/api/v0.1/LoginWithUserPass` endpoint

```json
{
  "username": "testuser",
  "password": "foo"
}
```

- You will receive a response:

```json
{
  "clientId": "...",
  "token": "...",
  "timeToLive": 86400
}
```

- In the [Swagger API](http://localhost:8080/swagger/index.html), use the Authorize
  button at the top of the page and put the `clientId` in the `VaultClientId` field
  and the `token` in the `VaultToken` field and press both Authorize buttons.
- You are now logged in. Try POSTing a new Case.

## Orchestrator Components

Components are available either in the global Docker registry,
or in the GitLab container registry:
https://gitlab.com/reductech/forensync/orchestrator/container_registry

### Consul

_port 8500_

Consul is a service networking platform that enables our services to detect one another.

```
docker run --publish 8500:8500 consul
```

Dashboard available at http://localhost:8500/ui/dc1/services
You can use the dashboard to see the running services and their health status.

### RabbitMQ

_port 5672_

RabbitMQ is the message broker we use to communicate between the services

```
docker run --publish 5672:5672 steeltoeoss/rabbitmq
```

Dashboard available at http://localhost:15672/#/ Username: _guest_ Password: _guest_
You can use the dashboard to see the exchanges and queues.

### MariaDB

_port 3306_

MariaDB is a relational database. We use it keep track of EvidenceContainers and Sequences.

```
docker run -e MYSQL_ROOT_PASSWORD=Steeltoe456 --publish 3306:3306 -d mariadb/server:10.3
```

_Login:_ Root, _Password:_ Steeltoe456

If you want to see the data, you can log in with any SQL administration tool (e.g. HeidiSQL)

### Spring Cloud Config

_port 8888_

Spring Cloud Config is a configuration server. It lets us define our service configurations
in a git repository and then provide minimal configuration in the services themselves.

You must use environment variables to give the location and credentials for your git repository

```
docker run -p 8888:8888 steeltoeoss/config-server -e spring.cloud.config.server.git.uri='https://gitlab.com/reductech/sandbox/spring-config-demo.git' -e spring.cloud.config.server.git.username='myusername' -e spring.cloud.config.server.git.password='mypassword'
```

### Datastore

The `datastore` service communicates with the database and updates evidence containers
and sequences when necessary.

```
docker run registry.gitlab.com/reductech/forensync/orchestrator/datastore
```

### HealthChecker

The `healthchecker` service monitors the worker queues and detects and issues with worker availability.

```
docker run registry.gitlab.com/reductech/forensync/orchestrator/healthchecker
```

### Sequencer

The `sequencer` service splits up sequences and decides which Workers to send which parts.

```
docker run registry.gitlab.com/reductech/forensync/orchestrator/sequencer
```

### Client API

The `Client API` is a service that provides endpoints for managing and monitoring sequences.

```
docker run -p 8080:80 registry.gitlab.com/reductech/forensync/orchestrator/clientapi
```

The API is available on port 8080.
Go to http://localhost:8080/swagger/index.html for the Swagger UI which will show
all the endpoints and let you test them.

### Worker

The `worker` service is responsible for actually running the sequences.
You may wish to have multiple workers with different configurations.
The worker service can be run either on-premise or in docker.

```
docker run registry.gitlab.com/reductech/forensync/orchestrator/worker
```

### Vault

Vault is an Authentication and Authorization service.

The API is available on port 8200
