# docker-nexus-digitalocean-deployment

## Goal

Build and deploy a private Docker-based application environment on a DigitalOcean Droplet, using Nexus Repository Manager as a private Docker registry and Docker Compose to run a Node.js application alongside MongoDB and Mongo Express.

## Prerequisites

This project assumes that a DigitalOcean Droplet has already been provisioned and is accessible through SSH.

The Droplet creation, SSH key configuration, firewall setup, and initial SSH connection are documented in the [digitalocean-java-app-deployment](https://github.com/saifaljuboori/digitalocean-java-app-deployment) repository.

### DigitalOcean Droplet Configuration

The Droplet used for this project was configured with:

* **Operating System:** Ubuntu 24.04.4 LTS
* **CPU:** 4 vCPUs
* **Memory:** 8 GB RAM
* **Nexus Web Interface:** 8081
* **Nexus Docker Registry:** 8082

For this demo environment, a Droplet with at least 2 vCPUs and 4 GB RAM is recommended.

## Project Scope

This project demonstrates an end-to-end workflow for building and deploying a containerized application using a self-hosted private Docker registry.

The project covers:

1. Installing and verifying Docker Engine and Docker Compose on the Droplet
2. Deploying Nexus Repository Manager as a Docker container with persistent storage
3. Configuring Nexus as a private Docker hosted registry
4. Configuring Docker clients to authenticate with the private registry
5. Building and publishing the Node.js application image to Nexus
6. Pulling the application image from Nexus on the DigitalOcean Droplet
7. Deploying the application stack with Docker Compose
8. Running the Node.js application alongside MongoDB and Mongo Express
9. Configuring application and database credentials through environment variables
10. Persisting MongoDB data using a Docker named volume
11. Verifying the complete deployment from an external browser

## Documentation

Each stage of the deployment is documented separately:

* [Docker Installation](docs/docker-installation.md)
* [Nexus Container Deployment](docs/nexus-container.md)
* [Private Docker Registry](docs/private-docker-registry.md)
* [Application Deployment](docs/application-deployment.md)

## Application Source

The Node.js application source code is maintained separately in the `docker-nodejs-mongodb` repository.

This repository focuses on the infrastructure and deployment side of the project, including Nexus Repository Manager, the private Docker registry, Docker Compose configuration, persistent storage, and deployment configuration.

The application image is built from the application source repository, published to Nexus, and then pulled by the DigitalOcean Droplet during deployment.

## Deployment Architecture

```text
Application Source
      |
      | Docker build
      v
Docker Image
      |
      | docker push
      v
Nexus Docker Hosted Registry
      |
      | docker pull
      v
DigitalOcean Droplet
      |
      | Docker Compose
      |
      +-------------------+
      |        |          |
      v        v          v
   Node.js  MongoDB  Mongo Express
    :3000    :27017      :8084
               |
               v
         Persistent Volume
```


