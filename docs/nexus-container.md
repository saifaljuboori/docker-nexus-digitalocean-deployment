# Run Nexus Repository Manager as a Docker Container

Nexus Repository Manager was deployed as a Docker container on the DigitalOcean Droplet.

## Pull the Nexus Docker Image

The official Nexus Repository Manager Docker image was pulled from Docker Hub:

```bash
docker pull sonatype/nexus3
```

Verify the image:

```bash
docker images
```

The `sonatype/nexus3` image should be listed.

## Create Persistent Storage

A Docker named volume was created to persist Nexus data independently from the container lifecycle:

```bash
docker volume create --name nexus-data
```

Verify the volume:

```bash
docker volume ls
```

The `nexus-data` volume should be listed.

## Run Nexus Repository Manager

The Nexus container was started using the `nexus-data` volume:

```bash
docker run -d -p 8081:8081 --name nexus -v nexus-data:/nexus-data sonatype/nexus3
```

This configuration:

* Runs Nexus in detached mode
* Maps Droplet port `8081` to container port `8081`
* Names the container `nexus`
* Mounts the `nexus-data` Docker volume to `/nexus-data`
* Uses the `sonatype/nexus3` Docker image

## Verify the Nexus Container

Check the running containers:

```bash
docker ps
```

The Nexus container should be running and expose port `8081`:

```text
0.0.0.0:8081->8081/tcp
```

## Monitor Nexus Startup

Nexus requires some time to initialize after the container starts.

View the container logs:

```bash
docker logs -f nexus
```

Once Nexus has completed its initialization, press `Ctrl+C` to stop following the logs. This does not stop the Nexus container.

## Retrieve the Initial Admin Password

The initial administrator password is generated during the first Nexus startup.

The password can be retrieved from inside the container:

```bash
docker exec nexus cat /opt/sonatype/sonatype-work/nexus3/admin.password
```

Use the returned password to log in to Nexus with the `admin` account.

## Access Nexus Repository Manager

Nexus Repository Manager is accessible through the Droplet's public IP address:

```text
http://<droplet-public-ip>:8081
```

After the first login, the initial administrator password was changed through the Nexus setup wizard.

## Verify Persistent Storage

The Nexus container uses the following Docker volume:

```text
nexus-data:/nexus-data
```

The volume allows Nexus data to persist independently from the lifecycle of the `nexus` container.

The volume can be inspected with:

```bash
docker inspect nexus
```

The container configuration confirms that the `nexus-data` Docker volume is mounted to `/nexus-data`.

