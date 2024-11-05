# Awesome Compose [![Awesome](https://awesome.re/badge.svg)](https://awesome.re)

 A curated list of Docker Compose samples.

 > **Note**
> The following samples are intended for use in local development environments such as project setups, tinkering with software stacks, etc. These samples must not be deployed in production environments.

## Samples of Docker Compose Applications

- [`RABBIT MQ`](./rabbit-mq) - Sample Rabbit MQ Service 

## Getting started

These instructions will get you through the bootstrap phase of creating and
deploying samples of containerized applications with Docker Compose.

### Prerequisites

- Make sure that you have Docker and Docker Compose installed
  - Windows or macOS:
    [Install Docker Desktop](https://www.docker.com/get-started)
  - Linux: [Install Docker](https://www.docker.com/get-started) and then
    [Docker Compose](https://github.com/docker/compose)
- Download some or all of the samples from this repository.

### Running a sample

The root directory of each sample contains the `docker-compose.yaml` which
describes the configuration of service components. All samples can be run in
a local environment by going into the root directory of each one and executing:

```console
docker compose up -d
```

To stop and remove all containers of the sample application run:
```console
docker compose down
```
---
 
> Check the `README.md` of each sample to get more details of service

