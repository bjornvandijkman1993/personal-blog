---
title: "Docker"
date: 2023-02-08T15:40:27+01:00
draft: true
---

In the world of microservices, it is important that your Docker images allow for quick deployment, compact size, and enhanced security. In this blog, I will show you how to create a secure and optimized Docker image for a Python application, using Poetry as the dependency manager. By utilizing the CLI tool Dive to examine the image and its layers, I will walk you through the steps to create an efficient Docker image. I wrote the Dockerfile to host a FASTAPI server for a machine learning application, but those concepts are out of scope of this blog. 
Please note that a basic understanding of Docker and Dockerfiles is assumed. For more information on the motivation to use Poetry instead of pip/pipenv/pip-tools/conda, read this and this blog.
```
FROM python:3.11-slim as build
```
I am using the slim version of Python 3.11 to minimize the size of the container and make it as lightweight as possible. It has a size of 121 MB.
The full Python Image (python:3.11) includes extra development tools and documentation that are not necessary for running the application, making it much larger with a size of 875 MB, around 7 times larger than the slim version 😱.

Another alternative is Alpine (python:3.11-alpine), which is smaller with a size of 56.5 MB, but lacks the package installer pip and support for installing wheel packages, which is needed for installing applications like Pandas and Numpy. To install these applications, you would need to compile them from source files using compiler packages like G++, which are also not installed by default on the Alpine image. This results in a larger image size (and a lot more hassle) than the slim version, so lets continue with that one. 

```
ENV PIP_DEFAULT_TIMEOUT=100 \
    # Allow statements and log messages to immediately appear
    PYTHONUNBUFFERED=1 \
    # disable a pip version check to reduce run-time & log-spam
    PIP_DISABLE_PIP_VERSION_CHECK=1 \
    # cache is useless in docker image, so disable to reduce image size
    PIP_NO_CACHE_DIR=1 \
    POETRY_VERSION=1.3.2
```

The ENV variables in the Dockerfile are set to optimize the behavior of pip and poetry during the installation of packages in the Docker container. I explicitly define the Poetry version to install.