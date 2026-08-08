# Configure Nexus as a Private Docker Registry

Nexus Repository Manager was configured to host private Docker images using a Docker hosted repository.

## Create Docker Hosted Repository

A Docker hosted repository named `docker-hosted` was created in Nexus.

The repository was configured with:

* **Repository type:** Docker (hosted)
* **Blob store:** default
* **Deployment policy:** Allow redeploy
* **Docker HTTP connector:** port `8082`

The Nexus web interface remains available on port `8081`, while Docker registry traffic uses port `8082`.

## Configure Repository Permissions

A dedicated Nexus role named `docker-repository-role` was created to provide access to the `docker-hosted` repository.

The role was assigned the following repository privileges:

* Browse
* Read
* Add
* Edit

These privileges allow the associated user to browse, pull, and push Docker images to the hosted repository.

## Create Docker Registry User

A dedicated Nexus user named `docker-user` was created for Docker registry access.

The user was assigned the `docker-repository-role`.

The dedicated account avoids using the Nexus administrator account for Docker registry operations.

## Configure DigitalOcean Firewall

An inbound firewall rule was added to allow TCP traffic on port `8082`.

The firewall was applied to the DigitalOcean Droplet hosting Nexus.

Port `8081` is used for the Nexus web interface, while port `8082` is used for the Docker registry connector.

## Publish the Docker Registry Port

The Nexus container was recreated with both the Nexus web interface and Docker registry ports published:

```bash
docker stop nexus
docker rm nexus

docker run -d \
  -p 8081:8081 \
  -p 8082:8082 \
  --name nexus \
  -v nexus-data:/nexus-data \
  sonatype/nexus3
```

The existing `nexus-data` Docker volume was reused so that Nexus configuration and repository data persisted when the container was recreated.

Verify the published ports:

```bash
docker ps
```

The Nexus container should expose both ports:

```text
0.0.0.0:8081->8081/tcp
0.0.0.0:8082->8082/tcp
```

## Verify Registry Connectivity

The Docker Registry API was tested using:

```bash
curl -v http://<droplet-public-ip>:8082/v2/
```

After completing the Nexus onboarding/EULA requirement, the registry endpoint became available for authenticated access.

## Configure Docker for HTTP Registry Access

The Nexus Docker connector was configured to use HTTP for this lab environment.

Because Docker normally expects HTTPS when communicating with registries, the Nexus registry address was added to Docker Desktop's `insecure-registries` configuration.

Open Docker Desktop → Settings → Docker Engine and add the following property to the existing JSON configuration:
```json
"insecure-registries": [
  "<droplet-public-ip>:8082"
]
```
Click Apply & Restart after saving the configuration.

Docker Desktop was then restarted.

Verify the configuration:

```bash
docker info
```

The registry should appear under:

```text
Insecure Registries:
```

## Authenticate with the Private Registry

Docker was authenticated using the dedicated Nexus user:

```bash
docker login <droplet-public-ip>:8082
```

The login completed successfully:

```text
Login Succeeded
```

## Push a Docker Image

The Alpine Linux image was used as a test image for the private registry.

Pull the image:

```bash
docker pull alpine:latest
```

Tag the image for the Nexus registry:

```bash
docker tag alpine:latest <droplet-public-ip>:8082/alpine:latest
```

Push the image:

```bash
docker push <droplet-public-ip>:8082/alpine:latest
```

The image was successfully pushed to Nexus.

## Verify the Image in Nexus

The pushed image was verified through:

**Nexus → Browse → docker-hosted**

The `alpine` image and its `latest` tag were visible in the hosted Docker repository.

## Result

The DigitalOcean Droplet now hosts a private Docker registry through Nexus Repository Manager.

The completed workflow is:

```text
Docker Client
     |
     | docker login
     | docker push
     v
DigitalOcean Droplet
     |
     | port 8082
     v
Nexus Docker Hosted Repository
     |
     v
alpine:latest
```

