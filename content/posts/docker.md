---
title: 'Maximize your Python Docker images: small, fast & secure'
date: 2023-02-08T15:40:27.000+01:00

cover:
    image: "docker.jpeg"
    alt: "Docker"
    relative: false

tags: ["docker", "security", "software engineering"]

---

In the world of **microservices**, it is important that your [**Docker**](https://www.docker.com) images allow for **quick deployment**, compact **size**, and enhanced **security**. In this blog, I will show you how to create a **secure** and **optimized** Docker image for a Python application, using [**Poetry**](https://python-poetry.org) as the dependency manager. By utilizing the CLI tool [**Dive**](https://github.com/wagoodman/dive) to examine the image and its layers, I will walk you through the steps to create an efficient Docker image. I wrote the Dockerfile to host a [FASTAPI](https://fastapi.tiangolo.com) server for a machine learning application, but those concepts are out of scope of this blog. 

Please note that a basic understanding of Docker and [Dockerfiles](https://docs.docker.com/engine/reference/builder/) is assumed. For more information on the motivation to use Poetry instead of pip/pipenv/pip-tools/conda, read [this](https://andrewbrookins.com/python/why-poetry) and [this](https://betterprogramming.pub/5-reasons-why-poetry-beats-pip-python-setup-6f6bd3488a04) blog.

```Docker
FROM python:3.11-slim as build
```

I am using the [**slim](https://hub.docker.com/layers/library/python/3.11-slim/images/sha256-1591aa8c01b5b37ab31dbe5662c5bdcf40c2f1bce4ef1c1fd24802dae3d01052?context=explore)** version of Python 3.11 to minimize the size of the container and make it as **lightweight** as possible. It has a size of **121** MB.

The full Python Image ([python:3.11](https://hub.docker.com/layers/library/python/3.11/images/sha256-79f7c6a20b4c513556c949c356ee075c4eb31aff8f64d8b81eac0799adc0e223?context=explore)) includes extra development tools and documentation that are not necessary for running the application, making it much larger with a size of **875 **MB, around **7 times** larger than the slim version 😱.

Another alternative is Alpine [python:3.11-alpine](https://hub.docker.com/layers/library/python/3.11-alpine/images/sha256-9f955672f82a7e7138bcb64bb8352fb0a3025533b8be90a789be5a32526497ca?context=explore), which is smaller with a size of **56.5** MB, but lacks the package installer pip and support for installing wheel packages, which is needed for installing applications like Pandas and Numpy. To install these applications, you would need to compile them from source files using compiler packages like [G++](https://www.geeksforgeeks.org/compiling-with-g-plus-plus/), which are also not installed by default on the Alpine image. This results in a larger image size (and a lot more hassle) than the slim version, so lets continue with that one.

```Docker
ENV PIP_DEFAULT_TIMEOUT=100 \
    # Allow statements and log messages to immediately appear
    PYTHONUNBUFFERED=1 \
    # disable a pip version check to reduce run-time & log-spam
    PIP_DISABLE_PIP_VERSION_CHECK=1 \
    # cache is useless in docker image, so disable to reduce image size
    PIP_NO_CACHE_DIR=1 \
    POETRY_VERSION=1.3.2
```

The ENV variables in the Dockerfile are set to optimize the behavior of pip and poetry during the installation of packages in the Docker container. I explicitly define the **Poetry** version to install.

## Security

![Photo by ](https://cdn-images-1.medium.com/max/15904/0*LcfOJjNZqamA3_B_)

Let’s move on to security, because there are a few aspects that are important. Running Docker containers as the **root** user is not recommended because the root user has complete control over the host system, including the ability to **modify** or **delete** files, start and stop services, and **access sensitive information**. To follow the **principle of least privilege**, it’s better to run the containers with only the minimum necessary privileges to run the application. To enhance security, we therefore create a non-root user for the Docker container named ‘appuser’.

```Docker
RUN set -ex \
    # Create a non-root user
    && addgroup --system appuser \
    && adduser --system --ingroup appuser --no-create-home appuser
```

Another security best-practice is updating and upgrading packages in a Docker container. The reason for this is that in a Docker environment, the base image used to create a container is typically a snapshot of a specific version of an operating system. Over time, security vulnerabilities or other issues may be discovered and patches released to address them using apt-get update and apt-get upgrade. The apt-get update command updates the package index, which is kind of like a database of available software packages and their metadata, and retrieves the latest package information from the package repository.

```Docker
RUN set -ex \
    && apt-get update \
    && apt-get upgrade -y
```

Now that we have information on the available software packages, we can use the apt-get upgrade command to upgrade the packages currently installed in the container to their latest available versions. The -y flag is used to automatically answer yes to any prompts during the upgrade process.

⬆ The update process resulted in some automatically installed packages, package cache files, and package index files. These files are no longer required when running the application after they have served their purpose of helping upgrade the packages. We can safely remove these files using the following commands, which will reduce the size of the Docker image.

```Docker
# Clean up
RUN set -ex apt-get autoremove -y \\
    && apt-get clean -y \\
    && rm -rf /var/lib/apt/lists/*
```

A final security best-practice is to make sure that no secrets are included in the docker image. To help prevent this, you can add common secrets files and folders to your **.dockerignore** file:

```Docker
**/.env
**/.aws
**/.ssh
```

You can also use [Trivy](https://github.com/aquasecurity/trivy), which will also warn you when you are for example ‘root’ as your image user. Here is a screenshot from the VSCode extension:

![VSCode extension for Trivy](https://cdn-images-1.medium.com/max/2054/1*dMoFS3WR84XuiYdCtrycMg.png)

## Application files

Alright, now that we have set some environment variables and improved the security of our docker container, lets continue to the copying of the actual application files and installing the dependencies. We want to set the working directory to ‘/app’. This directory does not exist yet in the base image, but WORKDIR creates it for us if it doesn’t exist.

```Docker
WORKDIR /app
COPY pyproject.toml poetry.lock ./
```

All subsequent instructions will be executed in this location, which makes our docker image more organized and portable.

The COPY command is used to copy files from the host system to the container file system. In this case, we are copying pyproject.toml and poetry.lock, which are the configuration files for the Poetry package manager.

## Dive

![Photo by ](https://cdn-images-1.medium.com/max/10944/0*Li3H1zhl_RUGaNm-)

Now that we have completed some steps, lets build the docker image and dive into it to inspect 🕵️‍♀️ the different layers:

```bash
docker build -t app:latest .
dive app:latest
```

👇 We can see in each layer which changes were made compared to the previous layer. First we created an **app** directory using the WORKDIR command, followed by the copying of two files.

![](https://cdn-images-1.medium.com/max/3844/1*0JS1YOLmAHyxByHETLACKg.png)

The current image has a size of **139 MB**. It is looking pretty good, but a couple of improvements can be made to decrease its size. The first thing to remember is that files that are removed by subsequent layers in the system are actually still present in the images; they’re just inaccessible in the final layer.

![](https://cdn-images-1.medium.com/max/2000/1*8M03e-cxUbCcjCvbQTvm5w.png)

👆Having the steps of installing the security updates and removing the now unnecessary files and packages separately therefore does not save any space. What does work is to combine these two steps into one, so lets do that and verify that this has decreased the size of the image.

```Docker
RUN set -ex \
    # Create a non-root user
    && addgroup --system appuser \
    && adduser --system --ingroup appuser --no-create-home appuser \
    # Upgrade the package index and install security upgrades
    && apt-get update \
    && apt-get upgrade -y \
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*
```

This decreased the size of the image with 18 mb to a total size of 121 mb 🎉, which is exactly equal to the apt-get update && update-get upgrade -y layer that we had before.

## Caching

Okay, lets continue! After installing the dependencies, we will copy some subdirectories that my application needs to the container. Followed by the installation of the python packages using Poetry:

```Docker
COPY ./artifacts artifacts
COPY ./api api

RUN pip install "poetry==$POETRY_VERSION" \
    && poetry install --no-root --no-ansi --no-interaction
```

🤔 But does the order of these commands make sense? Remember that each layer is an **independent delta** from the layer below it. Every time you change a layer, it changes every layer that comes after it. If the previous layer is exactly the same as before, we can just use the cached SHA value instead of rebuilding that step.

```bash
Step 12/18 : COPY ./api api
    ---> Using cache
    ---> ea3ba41d1a13
```

Therefore, you want to **order your layers** from least likely to change to most likely to change for fast building times💨. It makes sense to first install the dependencies before copying the rest of the program. You are generally much less likely to update and change your dependencies than your application code. 👇 This is therefore a better order of instructions:

```Docker
RUN pip install "poetry==$POETRY_VERSION" \
    && poetry install --no-root --no-ansi --no-interaction

COPY ./artifacts artifacts
COPY ./api api
```

The next step is to expose a port for our application to listen on. EXPOSE in a **Dockerfile** is more for **documentation** 📄 purposes, indicating which ports the application listens on within the container.

To actually expose our FASTAPI server, we run the following command:

```Docker
CMD ["uvicorn", "api.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

Finally, we set the user that the application will run as using the USER command. This is set to the user we created earlier, appuser.

Let’s build the docker image again and check out the size:

![](https://cdn-images-1.medium.com/max/2000/1*WsSCcueCagpBEixHMntibg.png)

The image now has a total size of **731** mb. **531** mb comes from the dependencies that were installed by Poetry, but Poetry itself also takes up some space.

💡 Although Poetry is useful during the development phase for creating virtual environments and managing dependencies, these functionalities are not required when running the Docker image, as the image provides its own isolated environment and our dependencies have already been installed. Reducing the size of the Docker image can therfore also be achieved by ensuring that the Poetry package is not included in the final image.

## Multistage builds

![](https://cdn-images-1.medium.com/max/2000/1*8r7gLDZCIMC6TtU27Omuvg.png)

Multistage builds can be utilized to exclude Poetry from the final Docker image, because it enables the creation of multiple images from a single Dockerfile. Instead of installing the dependencies directly using Poetry, Poetry can also export the necessary dependencies to a **requirements.txt **file during the build stage. This file can be copied to the final stage, and be used by the **pip** to install the dependencies.

```Docker
FROM python:3.11-slim as build

. . . 

RUN pip install "poetry==$POETRY_VERSION" \
    && poetry install --no-root --no-ansi --no-interaction \
    && poetry export -f requirements.txt -o requirements.txt


### Final stage
FROM python:3.11-slim as final

WORKDIR /app

COPY --from=build /app/requirements.txt .

RUN pip install -r requirements.txt
```

By excluding Poetry in the final stage, the size of the Docker image is reduced, as demonstrated by the decrease from 732 MB to 538 MB 🎉🍾.

When running the docker image as **appuser**, it was not allowed to create any new files in the image. However, my application creates a new file in a subdirectory called artifacts whenever the application is run. To give appuser the permission to create this file, explicit permission needs to be given:

```Docker
RUN chown -R appuser:appuser /app/artifacts
```

👇 We therefore end up with the following file:

```Docker
FROM python:3.11-slim as build

ENV PIP_DEFAULT_TIMEOUT=100 \
    # Allow statements and log messages to immediately appear
    PYTHONUNBUFFERED=1 \
    # disable a pip version check to reduce run-time & log-spam
    PIP_DISABLE_PIP_VERSION_CHECK=1 \
    # cache is useless in docker image, so disable to reduce image size
    PIP_NO_CACHE_DIR=1 \
    POETRY_VERSION=1.3.2

WORKDIR /app
COPY pyproject.toml poetry.lock ./

RUN pip install "poetry==$POETRY_VERSION" \
    && poetry install --no-root --no-ansi --no-interaction \
    && poetry export -f requirements.txt -o requirements.txt


### Final stage
FROM python:3.11-slim as final

WORKDIR /app

COPY --from=build /app/requirements.txt .

RUN set -ex \
    # Create a non-root user
    && addgroup --system appuser \
    && adduser --system --ingroup appuser --no-create-home appuser \
    # Upgrade the package index and install security upgrades
    && apt-get update \
    && apt-get upgrade -y \
    # Install dependencies
    && pip install -r requirements.txt \
    # Clean up
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*

COPY ./artifacts artifacts
COPY ./api api

EXPOSE 8000

# give permission to write files to the artifacts subdirectory
RUN chown -R appuser:appuser /app/artifacts

CMD ["uvicorn", "api.main:app", "--host", "0.0.0.0", "--port", "8000"]

# Set the user to run the application
USER appuser
```

## Summary

In conclusion, by following these tips and best practices when creating your Dockerfile, you can ensure that your final image is optimized for security, size and performance.

Make sure to choose the right base image, understand the immutability of Docker layers, put instructions in the right order, and follow the security best-practices.

Thanks for reading! Feel free to reach out if you found this post interesting or if it helped you out in any way.