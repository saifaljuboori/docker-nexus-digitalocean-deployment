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

