# Docker
Docker is a containerization tool written in Go to create lightweight containers running isolated file systems and processes.

## Installation
On Ubuntu, Docker can be installed with `apt`:
>sudo apt install docker.io

To install the latest version however, it is recommended to follow the official [Docker Installation](https://docs.docker.com/install/linux/docker-ce/ubuntu/) documentation:

```bash
sudo apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

The `docker` hello-world should be run to verify installation:
>sudo docker run hello-world

## Permissions
Docker requires super user permissions and must be run with the `sudo` command by default. This can be bypassed for convenience by adding your user to the docker unix group:
>sudo usermod -aG docker <USER>

Log out, then log back in to see the changes. If successful, the following command will now work:
>docker run hello-world

## Example: Dockerizing your Go project
Building and running your Go programs in its own isolated docker container is a simple matter of creating a `Dockerfile` script in the project directory with some basic commands:
```Dockerfile
FROM golang
COPY . .
RUN go build main.go
ENTRYPOINT ["/your-app-name"]
```

However, this will create a large monolithic image file that may not be suitable for future deployments across a network. Much more recommended is to create a multi-stage Docker build:
```Dockerfile
# Starting with the official golang/alpine image, tagged as our builder
FROM golang:alpine AS builder
# Install git for go getting any dependencies
RUN apk update && apk add --no-cache git
# Create a workspace in our container and copy all files in the Dockerfile directory
WORKDIR $GOPATH/src/your-program-path/
COPY . .
# Install any dependencies
RUN go get -d 
# Compile
RUN go build main.go
​
# Create a new image for our compiled application
FROM scratch
# Copy from our builder
COPY --from=builder $GOPATH/src/your-program-path/your-app-name your-app-name
ENTRYPOINT ["/your-app-name"]
```

## Example: PostgreSQL Server
To run a simple Postgres server instance:
>docker run -p [port]:5432 postgres

Where [port] is your choice of port. A common port is 5432, a default for many Postgres databases:
>docker run -p 5432:5432 postgres

To run a database as a background process, use the `-d` switch:
>docker run -d -p 5432:5432 postgres

It's a good idea to label a database with the `--name` switch:
>docker run -d --name my_postgres -p 5432:5432 postgres

To see running containers, use:
>docker ps

To stop a container, run:
>docker stop <CONTAINER-NAME-OR-ID>

# Running PostgreSQL with Docker & a Dockerfile
To create a custom postgres database, create a file named `Dockerfile`:
```Dockerfile
FROM postgres:10
ENV POSTGRES_USER hello-postgres
ENV POSTGRES_PASSWORD hello-postgres
ADD init.sql /docker-entrypoint-initdb.d
EXPOSE 5432
```

Where init.sql is an optional `.sql` script file in the same directory as your Dockerfile. Use `ENV` to set the username and password as needed. Both `ENV` lines can be omitted, causing the username and password to default to `postgres`

To build an image from the Dockerfile:
>docker build -t demo-postgres .

Where `demo-postgres` is the name you wish to give this image. Then to run the build:
>docker run -p 5432:5432 -d demo-postgres

Remember to expose the ports and use any desired flags.

# Connecting to a Docker PostgreSQL database
With your Docker postgres image built and a container running, use `docker exec` to run the `psql` tool on the running container:
>docker exec -it demo-postgres psql -U hello-postgres

Enter your Postgres username after `-U` (`postgres` by default) and enter your password when prompted.