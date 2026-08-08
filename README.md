# docker-nexus-digitalocean-deployment

## Goal

Run Nexus Repository Manager as a Docker container on a DigitalOcean Droplet.

## Prerequisites

This project assumes that a DigitalOcean Droplet has already been provisioned and is accessible through SSH.

The DigitalOcean Droplet creation, SSH key configuration, firewall setup, and initial SSH connection are documented in the [digitalocean-java-app-deployment](https://github.com/saifaljuboori/digitalocean-java-app-deployment) repository.

### DigitalOcean Droplet Configuration

The Droplet used for this project was configured with:

* **Operating System:** Ubuntu Linux
* **CPU:** 4 vCPUs
* **Memory:** 8 GB RAM
* **Nexus Repository Manager Port:** 8081

For this demo environment, a Droplet with at least 2 vCPUs and 4 GB RAM is recommended to provide sufficient resources for Docker and Nexus Repository Manager.

## Project Scope

Once the Droplet is available, this project covers:

1. Installing Docker Engine on the Droplet
2. Pulling the Nexus Repository Manager Docker image
3. Configuring persistent Nexus storage
4. Running Nexus Repository Manager as a Docker container
5. Verifying access to Nexus Repository Manager

## Application Deployment

The final deployment stage is documented in [`docs/application-deployment.md`](docs/application-deployment.md).

This stage covers:

1. Preparing the Node.js application for external deployment
2. Updating frontend API requests to use relative paths
3. Building the `my-app:1.1` Docker image
4. Publishing the image to the private Nexus Docker registry
5. Pulling the application image from Nexus on the DigitalOcean Droplet
6. Configuring environment variables for the Docker Compose deployment
7. Running the Node.js application, MongoDB, and Mongo Express as a Docker Compose stack
8. Verifying the deployed application and Mongo Express from an external browser

The application source code is maintained separately in the `docker-nodejs-mongodb` repository. This repository focuses on the infrastructure, Nexus registry, and deployment configuration.

