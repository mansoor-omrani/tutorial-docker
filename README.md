# Docker Container Tutorial

## What is Docker?
It's a way to package software to run in any hardware.

## Concepts
- Dockerfile
- Docker Images
- Containers
- Volumes
- Container Registry

### Dockerfile
- It's just a text file.
- It's a blueprint to create a docker image.
- It defines an environment where our software can run on.
- Any developer can use the dockerfile to rebuild the environment.

### Image
- It's a template to run containers.
- It's an *immutable*.
- It contains a tiny self-hosted OS with all its files plus your software.
- It can be uploaded to public/private image registries.
Any developer can pull the image to create another image from that.
- This way, the docker image can be run multiple times in multiple places.

a Docker Image contains ...
- OS
- Runtime environment
- Application
- Dependencies
- Configurations
- Tools
- Commands

An image made up from a parent image.

### Container
- It's a docker image that is running in a process.
- Tools like Kubernetes or Swarm come into play to scale containers to an infinite workload.
- Each container has an ID that is linked to a docker image.
- An image can be run multiple times, resulting in multiple running containers.

### Container Registry
A registry that hosts images and provides various features, such as:
- encryption
- compression
- control access
- version
- tag
- manage lifecycle
- ...

example: `Docker Hub`

## Dockerfile
It's just a plain text file.

example:

```dockerfile
FROM node:12-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

ENV PORT=8080

EXPOSE 8080

CMD ["npm", "start"]
```

## .dockerignore
It's a text file so that docker ignores some files.

example:
```
node_modules
```

**Note**: Do not add `dist` or `build` to `.dockerignore`. That way, docker will ignore commands targeting those folders.
For example, your `npm start` command that runs `node dist/index.js` internally, will not work any longer, since `dist` folder created by a `npm build` will be ignored.

## Docker Image
### Build
syntax:
```
docker build -t {image-name} {dockerfile-path}
```

example:
```
docker build -t myapp:1.0 .
```

**Attention**: image names are unique. If you use an existing name, the existing image will be renamed to `none:none`
and will be in dangling state. It is a good practice to use version in image names after colon sign and increase version upon building.

### Run
syntax:
```
docker run -p {local port}:{docker exposed port} {image-id-or-tag}
```

example:
```
docker run -p 5000:8080 b9094ddfe920
```

**Note**: Running a docker image creates a new container.

**Note**: What makes `Docker Desktop` a little confusing is that there are two sets of `Run` buttons. One in `Images`, the other in `Containers`. The first one is to create a new container. The other is to run an existing container.

### Rename
```
docker image tag {old:name} {new:name}
```

### Remove
```
docker image rm <IMAGE ID>
docker rmi <IMAGE ID>
```

**Note**: There should not be any running containers for the image being deleted. To force delete, you should use `-f` switch.

```
docker image rm <IMAGE ID> -f
```

### Update
There is no mechanism to update an existing image. You should remove the old one and build a new image with the same name.

### Save to tar
```
docker save {image} > {tar-file-name}
```

```
docker save my-nodejs-app-image > img.tar
```

### Load and create from a tar
```
docker load -i {tar-file-name}
```

## Container
Containers are a bit confusing. So, it is best to understand what a container in reality is.

A container is ...
- a copy of an image, ...
- giving it a name, ...
- and storing a command for running it, ...
- so that you can later easily start/stop it.

### Create a new Container
```
docker run --name <container_name> <image_name>
```

### Specify port
Map a container’s exposed port(s) to a port in the host.

```
docker run -p <host_port>:<container_port> <image_name>
```

### Run in background (daemon)
Run a container in the background:

```
docker run --name <container_name_or_id> -d <image_name>
```

### Delete on exit
```
docker run --name <container_name_or_id> -d <image_name> --rm
```

### Start/Stop
Start or stop an existing container:

```
docker start|stop <container_name_or_id>
```

### Rename
```
docker rename <old_name> <new_name>
```

### Shell
Open a shell inside a running container:
```
docker exec -it <container_name_or_id> sh
```

**Note**: If you change a file in a running container, your change is persisted, even when you stop the container and start it again. These changes are not applied to the image the container was created from.

### Logs
Fetch and follow the logs of a container:
```
docker logs -f <container_name_or_id>
```

### Show ports
```
docker port {container-name-or-id}
```

### List
list currently running containers:
```
docker ps
```

result:

```
| CONTAINER ID | IMAGE | COMMAND | CREATED | STATUS | PORTS | NAMES |
0839bb6b6e4c	docker-demo-web	"docker-entrypoint.s..."	21 hours ago	Up 21 hours	0.0.0.0:8080->8080	docker-demo-web_1
```

List all docker containers (running and stopped):
```
docker ps --all
```

### Copy file
Copy a file from the host to a container
```
docker cp {source-file} {container-id}:{target-file}
```

example:
```
docker cp c:\temp\a.txt my-node-app-container:/app/a.txt
```

### Remove
```
docker rm <CONTAINER ID>
```

## Persisting and sharing Data
When you stop a container, its process is stopped and everything in the process memory is lost. So, we have to persist data outside of containers. Data could be anything that should not be disposed when container is stopped. Examples are databases, data files, excel/csv files, etc.

There are two mechanism for persisting data outside containers:

- `Volumes`
- bind `Mounts`

Using these two mechanisms we can also share data among multiple containers.

### Bind local directory
Using `-v` or `--volume` switch when running an image `docker run` we can mount a local directory to a directory in the container being created and run.

```
docker run --volume=C:\docker-share:/share -p 5001:8080 -d fireship/demoapp:1.3

docker run -v C:\docker-share:/share -p 5001:8080 -d fireship/demoapp:1.3
```

This is a 2-way sync. i.e., if you change anything in mounted directory from the host, container can see it and vice versa. If container changes anything inside that directory, the host can see it.

You can also use `--mount` switch as well.
```

docker run --mount source=C:\docker-share,target=/share -p 5001:8080 -d fireship/demoapp:1.3
```

### --volume vs --mount
In general, `--mount` is more explicit and verbose. The biggest difference is that the `-v` syntax combines all the options together in one field, while the `--mount` syntax separates them.

`-v` or `--volume`: Consists of three fields, separated by colon characters (:). The fields must be in the correct order.

- In the case of bind mounts, the first field is the path to the file or directory on the host machine.
- The second field is the path where the file or directory is mounted in the container.
- The third field is optional, and is a comma-separated list of options, such as ro, z, and Z.

`--mount`: Consists of multiple key-value pairs, separated by commas and each consisting of a <key>=<value> tuple. The order of the keys is not significant.

#### Types of bind
- `bind`
- `volume`
- `tmpfs`.

#### Keys
- `source` or `src`: it is the path to the file or directory on the host.
- `target` or `dst` or `destination`: it is the directory in the container.
- `readonly`: if present, makes host directory/file read-only to container.
- `bind-propagation`: if present, changes the bind propagation. May be one of `rprivate`, `private`, `rshared`, `shared`, `rslave`, `slave`.

The --mount flag does not support z or Z options for modifying selinux labels.

### Volumes
Volumes have the same idea as mounting. A volume is just a folder on the host machine that is mapped to a folder in container.

The advatnage of volume over mounting is that it does not depend on certain operating system specs.

You do not need to know where the real directory in the host is located. It is not even necessary that the directory already existed. Docker created it on demand if not already existed and manages it on behalf of you.

You can see the content of volume using `Docker Desktop`.

#### Create volume
```
docker volume create my-share

docker run -v my-share:/share -p 5001:8080 -d fireship/demoapp:1.3
```

#### List volumes
```
docker volume ls
```

#### Inspect volume
```
docker volume inspect my-share

[
    {
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/my-share/_data",
        "Name": "my-share",
        "Options": {},
        "Scope": "local"
    }
]
```

#### Back up a volume
```
docker run -v /my-volume --name my-container my-image

docker run --rm --volumes-from my-container -v /backup my-image tar cvf /backup/backup.tar /my-volume
```

#### Remove volume
```
docker volume remove my-share
```

## Mount into a non-empty directory on the container
If you bind-mount a directory into a non-empty directory on the container, the directory's existing contents are obscured by the bind mount. This can be beneficial, such as when you want to test a new version of your application without building a new image. However, it can also be surprising and this behavior differs from that of volumes.

## Recursive mounts
When you bind mount a path that itself contains mounts, those submounts are also included in the bind mount by default. This behavior is configurable, using the `bind-recursive` option for `--mount`. This option is only supported with the `--mount` flag, not with `-v` or `--volume`.

## Container Registry
```
docker push
docker pull
```

## Advanced Topics
### Prune
Delete all images and containers
```
docker system prune
```

### Docker Compose
Usually your app requires multiple containers. It is cumbersome to manually create/start/stop containers. Docker compose is a tool to help with this need. It's a tool to run multiple containers at the same time, such as:

- database
- back-end api
- front-end app

or an app with multiple microservices.

docker-compose.yaml example:
```yml
version: '3'
services:
  web:
    build: .
    container_name: nodeapp_c
    ports:
      - "8080:8080"
  db:
    image: "mysql"
    environment: 
      MYSQL_ROOT_PASSWORD: password
    volumes:
      - db-data:/foo

volumes:
  db-data:
```

Starting containers:
```
docker-compose up
```

Stopping and Deleting containers:
```
docker-compose down
```

Stopping and Deleting containers and images:
```
docker-compose down --rmi all
```

Stopping and Deleting containers, images and volumes:
```
docker-compose down --rmi all -v
```
