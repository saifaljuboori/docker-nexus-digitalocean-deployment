# Deploy the Node.js Application with Docker Compose

This branch completes the application deployment by running the Node.js application, MongoDB, and Mongo Express as a Docker Compose stack on the DigitalOcean Droplet.

The Node.js application source code is maintained separately in the `docker-nodejs-mongodb` repository. This deployment project references the resulting Docker image rather than duplicating the application source code.

## Application Source

The application used in this deployment is maintained in the following repository:

* `docker-nodejs-mongodb`

The application repository contains:

* Node.js application source code
* Frontend files
* `Dockerfile`
* `package.json`
* `package-lock.json`

The Docker image for this deployment was built from that application.

## Deployment-Specific Frontend Change

Before deploying the application to the DigitalOcean Droplet, the frontend API requests were updated.

Originally, the frontend used absolute URLs pointing to:

```text
http://localhost:3000
```

For example:

```javascript
fetch('http://localhost:3000/get-profile')
```

and:

```javascript
fetch('http://localhost:3000/update-profile', {
```

This does not work correctly when the application is accessed from an external browser because `localhost` refers to the computer running the browser, not the DigitalOcean Droplet.

The frontend was therefore changed to use relative API paths:

```javascript
fetch('/get-profile')
```

and:

```javascript
fetch('/update-profile', {
```

With this configuration, requests are sent to the same host and port from which the frontend was loaded.

For example:

```text
http://<droplet-public-ip>:3000/get-profile
http://<droplet-public-ip>:3000/update-profile
```

This change was included in the new Docker image deployed by this branch.

## Build the Application Image

After updating the frontend, a new Docker image was built.

The application image version used for the final deployment is:

```text
my-app:1.1
```

The image was verified locally:

```bash
docker images | grep my-app
```

The image was then prepared for the private Nexus Docker registry.

## Tag the Image for Nexus

The image was tagged using the Nexus Docker hosted repository:

```bash
docker tag my-app:1.1 159.65.112.17:8082/my-app:1.1
```

The resulting image reference is:

```text
159.65.112.17:8082/my-app:1.1
```

## Push the Application Image to Nexus

The Docker image was pushed to the Nexus Docker hosted repository:

```bash
docker push 159.65.112.17:8082/my-app:1.1
```

The `1.1` image was verified in Nexus under the `docker-hosted` repository.

The Nexus registry therefore acts as the private image repository between the development environment and the DigitalOcean deployment environment.

## Configure the Droplet for the Nexus HTTP Registry

The DigitalOcean Droplet communicates with the Nexus Docker registry using HTTP for this lab environment.

Docker was configured through:

```text
/etc/docker/daemon.json
```

The registry was added as an insecure registry:

```json
{
  "insecure-registries": [
    "159.65.112.17:8082"
  ]
}
```

After the Docker configuration was applied, the Droplet could communicate with the Nexus Docker registry.

## Authenticate with Nexus from the Droplet

The Droplet was authenticated against the private Docker registry:

```bash
docker login 159.65.112.17:8082
```

The login completed successfully.

## Pull the Application Image

The application image was then pulled directly from Nexus:

```bash
docker pull 159.65.112.17:8082/my-app:1.1
```

The image was verified locally on the Droplet:

```bash
docker images | grep my-app
```

The image ID matched the image that was pushed to Nexus.

## Docker Compose Configuration

The deployment uses:

```text
docker-compose.yaml
```

The Compose stack contains three services:

```text
my-app
mongodb
mongo-express
```

The application service references the image stored in Nexus:

```yaml
my-app:
  image: 159.65.112.17:8082/my-app:1.1
```

The application exposes port `3000`:

```yaml
ports:
  - 3000:3000
```

MongoDB exposes port `27017` and uses a named Docker volume for persistent database storage:

```yaml
volumes:
  - mongo-data:/data/db
```

Mongo Express is exposed through host port `8084`:

```yaml
ports:
  - 8084:8081
```

Port `8084` is used because port `8081` on the Droplet is already used by Nexus Repository Manager.

## Environment Variables

Sensitive configuration values were removed from the Compose file and replaced with environment-variable references.

The Compose file uses variables for:

```text
MONGO_DB_USERNAME
MONGO_DB_PWD
MONGO_INITDB_ROOT_USERNAME
MONGO_INITDB_ROOT_PASSWORD
ME_CONFIG_MONGODB_ADMINUSERNAME
ME_CONFIG_MONGODB_ADMINPASSWORD
ME_CONFIG_BASICAUTH_USERNAME
ME_CONFIG_BASICAUTH_PASSWORD
```

The actual values are provided through a `.env` file on the DigitalOcean Droplet.

The `.env` file is deployment-specific and is not committed to the Git repository.

The Compose configuration can therefore reference the variables without storing the actual credentials in source control.

## Validate the Compose Configuration

Before starting the stack, the Compose configuration can be validated with:

```bash
docker compose -f /root/docker-compose.yaml config
```

This verifies the Compose file and resolves the environment-variable configuration.

## Deploy the Application Stack

The Compose stack was started on the DigitalOcean Droplet with:

```bash
docker compose -f /root/docker-compose.yaml up -d
```

The stack creates and starts:

```text
my-app
mongodb
mongo-express
```

MongoDB data is stored in the Docker named volume:

```text
mongo-data
```

## Verify the Running Stack

The Compose services can be checked with:

```bash
docker compose -f /root/docker-compose.yaml ps
```

The final deployment exposes:

```text
my-app          0.0.0.0:3000->3000/tcp
mongodb         0.0.0.0:27017->27017/tcp
mongo-express   0.0.0.0:8084->8081/tcp
```

The Nexus container continues to expose:

```text
8081 -> Nexus web interface
8082 -> Nexus Docker registry
```

## Final Application Verification

The deployed Node.js application was verified from an external browser:

```text
http://<droplet-public-ip>:3000/
```

The application loaded successfully and its frontend API requests worked correctly.

Mongo Express was also verified:

```text
http://<droplet-public-ip>:8084/
```

Both services were successfully accessible from outside the Droplet.

## Final Deployment Architecture

The completed environment consists of Nexus and the application stack running on the same DigitalOcean Droplet:

```text
                         DigitalOcean Droplet
                                  |
              +-------------------+-------------------+
              |                                       |
              |                                       |
        Nexus Repository                        Docker Compose
              |                                       |
      +-------+-------+                    +----------+----------+
      |               |                    |          |          |
   :8081            :8082               my-app     MongoDB   Mongo Express
      |               |                    |          |          |
 Nexus UI       Docker Registry          :3000     :27017     :8084
                                     
                                     
Docker Client
      |
      | docker push
      v
Nexus Docker Hosted Repository
      |
      | docker pull
      v
DigitalOcean Droplet
      |
      v
my-app:1.1
```

## Deployment Flow

The final application deployment workflow is:

```text
Node.js application source
        |
        | Frontend deployment change
        | localhost → relative API paths
        v
Docker build
        |
        | my-app:1.1
        v
Nexus Docker Hosted Repository
        |
        | docker pull
        v
DigitalOcean Droplet
        |
        | docker-compose.yaml
        v
+-----------------------------+
| my-app                      |
| mongodb                     |
| mongo-express               |
+-----------------------------+
        |
        v
External browser
        |
        +--> Application :3000
        |
        +--> Mongo Express :8084
```

## Result

The final branch completes the deployment of the Node.js application alongside MongoDB and Mongo Express.

The application image is stored in the private Nexus Docker registry, the Droplet pulls the image from Nexus, and Docker Compose manages the complete application stack.

The deployment also uses environment variables for credentials and persistent Docker storage for MongoDB data, while the frontend uses relative API paths so that the application functions correctly when accessed through the Droplet's public IP address.

