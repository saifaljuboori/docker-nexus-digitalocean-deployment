# Docker Installation

Docker Engine and Docker Compose are required to run Nexus Repository Manager as a Docker container.

Docker was installed on the DigitalOcean Droplet and verified before proceeding with the Nexus setup.

## Environment

- **Operating System:** Ubuntu 24.04.4 LTS
- **Docker Engine:** 29.7.1
- **Docker Compose:** v5.3.1

## Verify Docker Installation

Check the Docker Engine version:

```bash
docker --version
```
Expected output:
```
Docker version 29.7.1, build e9452d6
```
Check Docker Compose:
```
docker compose version
```
Expected output:
```
Docker Compose version v5.3.1
```
Docker is now ready to run the Nexus Repository Manager container.
