# Docker is a platform that helps us build, run and share software.

<!-- TOC -->

- [Docker is a platform that helps us build, run and share software.](#docker-is-a-platform-that-helps-us-build-run-and-share-software)
  - [Containers](#containers)
  - [Images](#images)
- [Search Images](#search-images)
- [Pull images](#pull-images)
- [Run Container](#run-container)
- [Expose Port](#expose-port)
- [Docker Compose](#docker-compose)
  - [Problem](#problem)
  - [Solution](#solution)
- [Set Jupyter Token](#set-jupyter-token)
  - [Problem](#problem)
  - [Solution](#solution)
- [Mount Drive](#mount-drive)
- [Dockerfile](#dockerfile)
- [Copy Files to Image](#copy-files-to-image)
- [Prune Containers](#prune-containers)
- [Push to DockerHub](#push-to-dockerhub)
  - [Rename the local repository](#rename-the-local-repository)
- [Remove Images](#remove-images)

<!-- /TOC -->

https://www.docker.com/

## Containers

> With containers, we can work on the same environment from different computers.

Solution for:

- Mac vs Windows vs Linux

- NumPy 1.25.1 vs NumPy 1.26.6 vs NumPy 1.23.0

## Images

> Docker images are similar to GitHub repositories. But instead of software, they also store `environments` where software is installed.

- Docker images have a read-only format. We can't modify it but create a new one.

- We use images (blueprints) to create containers (houses).

- Containers are running instances of images. Unlike images, we can change and interact with them.

<br />

# 1. Search Images

```
docker search tensorflow
```

# 2. Pull images

```
docker pull jupyter/tensorflow-notebook
```

- `docker pull jupyter/tensorflow-notebook`: repository name

  https://hub.docker.com/r/jupyter/tensorflow-notebook

- `jupyter`: the name of the community who maintain this image

- `tensorflow-notebook`: the name of the image itself

# 3. Run Container

```
docker run jupyter/tensorflow-notebook

```

And we get the URLs:

```
http://692eb16ba958:8888/lab?token=067c7f0d77be0b1d0f2a62d461fa5963fa087b0cccd7069b
http://127.0.0.1:8888/lab?token=067c7f0d77be0b1d0f2a62d461fa5963fa087b0cccd7069b
```

Then

```
ctrl + c
```

# 4. Expose Port

```
docker run -p 8000:8888 jupyter/tensorflow-notebook
```

- `8000`: `external port of host system` We can choose this port.

- `8888`: `internal port of container` Fixed. Need to refer to the above URLs.

And we get new URLs:

```
http://99928ac600a0:8888/lab?token=7c77d45cd10d365f95fddd8d1c6648954be4dc6581143d96
http://127.0.0.1:8888/lab?token=7c77d45cd10d365f95fddd8d1c6648954be4dc6581143d96
```

Then, Go to `localhost:8000`

Copy and paste the token from the above URLs.

# 5. Docker Compose

## Problem

If we try to:

```
from transformers import pipeline
```

It returns an error because transformers is not installed in this container.

Please note the whole idea of containers is we never need to install anything. Thus, if something is missing from our image, we cannot just add it by running `pip install`. Instead, we need to build a brand-new image!

## Solution

Luckily we don't need to build it from scratch. We can use `jupyter/tensorflow-notebook` as a base and combine it with a new module.

Reproduce the same container with a `compose.yml` that defines it.

> `.yml` language: indented pair of keys and values separated by `:`

```
---
services:
    transformers-notebook:
        image: jupyter/tensorflow-notebook
        ports:
            - 8000:8888
...

```

Open the folder containing the above `compose.yml` file in terminal, and run:

```
docker compose up
```

We will get new URLs and tokens.

```
transformers-notebook-1  | [I 2024-03-19 03:12:49.369 ServerApp] http://9afb69596df1:8888/lab?token=73f323bdcce268fd487c478fe8167590fa620c695704df51
transformers-notebook-1  | [I 2024-03-19 03:12:49.369 ServerApp]     http://127.0.0.1:8888/lab?token=73f323bdcce268fd487c478fe8167590fa620c695704df51
```

# 6. Set Jupyter Token

## Problem

Set a password, instead of copying and pasting token every time

## Solution

Shut down and remove the container

```
ctrl + c

docker compose down
```

Modify the `compose.yml`

```
---
services:
    transformers-notebook:
        image: jupyter/tensorflow-notebook
        ports:
            - 8000:8888
        environment:
            - JUPYTER_TOKEN=iambatman
...
```

# 7. Mount Drive

> A way to preserve our files

We expose a folder from our container to a folder on our host machine.

```
---
services:
    transformers-notebook:
        image: jupyter/tensorflow-notebook
        ports:
            - 8000:8888
        environment:
            - JUPYTER_TOKEN=iambatman
        volumes:
            - ./:/home/jovyan
...
```

`/home/jovyan`: refer to the original image

Now, `docker compose up`, and we can see `compose.yml` in our file tree.

# 8. Dockerfile

Build a new image, instead of using an existing image (jupyter/tensorflow-notebook)

```
---
services:
    transformers-notebook:
        build: ./
        ports:
            - 8000:8888
        environment:
            - JUPYTER_TOKEN=iambatman
        volumes:
            - ./:/home/jovyan
...
```

`build`: specify the location of the `Dockerfile`

Create a `Dockerfile` in the same directory where `compose.yml` locates and edit it:

```
# Specify the parent image
FROM jupyter/tensorflow-notebook

# Specify the user with the right permissions
# If nor sure -> USER root
USER $NB_UID

RUN pip install --upgrade pip && \
    pip install transformers && \
    pip install pysrt && \
    fix-permissions "/home/${NB_USER}"
```

Again, `docker compose up`, and we can import `transformers`.

# 9. Copy Files to Image

Convert the entire software (environment + code) into an image

```
FROM jupyter/tensorflow-notebook

USER $NB_UID

RUN pip install --upgrade pip && \
    pip install transformers && \
    pip install pysrt && \
    fix-permissions "/home/${NB_USER}"

# Adding wanted files to the root folder
COPY captions_english.srt translator.ipynb ./
```

# 10. Prune Containers

```
ctrl + c

# remove all the stopped containers
# but `docker compose down` also works
docker container prune

# Build our new image
docker compose up
```

# 11. Push to DockerHub

DockerHub -> Repository -> Create

## Rename the local repository

```
# Find all local repositories
docker images

# Rename it
# docker image tag <oldName:tag> <nameOnHub:version>
docker image tag transformers-notebook-transformers-notebook:latest weiweiyeih/srt-translator:1.0

# Double check
docker images

# login
docker login

# Push up to Hub
docker push weiweiyeih/srt-translator:1.0
```

# 12. Remove Images

```
# Remove stopped containers
docker container prune

# Remove local instances of images
docker rmi weiweiyeih/srt-translator:1.0

# Double check it's gone
docker images

# Pull the image
docker pull weiweiyeih/srt-translator:1.0

# Run it
# Again, you can choose to use `9000` or another port
docker run -p 9000:8888 weiweiyeih/srt-translator:1.0

```

Copy the returned URL and change the port to `9000`

```
http://127.0.0.1:9000/lab?token=9f251b526236f52483d9efa83d1c0916a38a5857dd4a23d2
```
