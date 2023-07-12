---
title: "Makefiles for Data Professionals"
date: 2023-02-16T09:37:49+01:00
draft: true

cover:
 image: "images/cover.jpeg"
 alt: "makefiles"
 relative: true
---

**Makefiles** were originally used to determine which parts of a large program need to be recompiled. Mostly for **C** or **C++** files. However, the **Make** utility can be used for more than just compilation. It can also be used to run a **sequence of instructions** or tasks. Additionally, it can serve as **documentation** 📄 for a project since the CLI commands that one would use to interact with the repository are explicitly defined in the Makefile.

In this blog, I will first explain the basics of the Makefile, and then go into more detail about how I specifically use it as a data engineer. 

### Anatomy of a Makefile
A typical Makefile consists of a series of rules, each of which defines how to build a particular **target** (i.e., a file or set of files). Each rule has the following structure:

```Make
targets: dependencies
    command
    command
    command
```

- **target**: The name of the file or target that is being built.
- **dependencies**: The list of files that the target depends on (i.e., the files that need to be compiled or processed to build the target).
- **command**: The shell command that should be executed to build the target.


### Makefile as a series of instructions
As a data engineer, I typically use Makefiles to run a sequence of instructions. For example, I might have a rule that builds and pushes a Docker image to a Docker registry. The rule would look like this:

```Make
build-and-push-image:
    docker build -t my-image .
    docker tag my-image:latest my-image:latest
    docker push my-image:latest
```

In this case, the target is **build-and-push-image**. This time, the target is not a file, but rather a name that is used to refer to a series of instructions. The command is a series of shell commands that build the image, tag it, and push it to a Docker registry.

### Phony targets
Since the rule in this Makefile does not build a file, it is called a "phony target". By default, Make assumes that all targets are files. However, you can explicitly state that a target is not a file by using the .PHONY directive. For example:

```Make
.PHONY: build-and-push-image
```

**Make** now knows **not** to check whether there are any files with the name **build-and-push-image**. There are a few reasons why you would want to do this:
- It may avoid **unnecessary builds** if a file already exists with the same name as the target. 
- It could also **prevent conflicts** if a file with the same name as the target is created in the future.
- Lastly, another reason to set a target as phony is to **document** the target. By setting a target as phony, it is clear that the target is not a file.

### Variables
Makefiles can also use **variables**. For example, you can define a variable for the name of the image:

```Make
IMAGE_NAME=my-image

build-and-push-image:
 docker build -t $(IMAGE_NAME) .
 docker tag $(IMAGE_NAME):latest $(IMAGE_NAME):latest
 docker push $(IMAGE_NAME):latest
```

In this case, the variable is called **IMAGE_NAME**. The variable is referenced by using the **$(IMAGE_NAME)** syntax. You can also define variables in the command line when running Make. We can define the tag of the image as a variable:

```bash
make build-and-push-image TAG=1.0.0
```

With the Makefile being:
```Make
.PHONY: build-and-push-image

build-and-push-image:
 docker build -t $(IMAGE_NAME):$(TAG) .
 docker tag $(IMAGE_NAME):$(TAG) $(IMAGE_NAME):$(TAG)
 docker push $(IMAGE_NAME):$(TAG)
```

### Functions
In my Makefile, I also created rules for running a Python file with a specific argument. I either want to run the file with the argument "train" or "predict". This is how I first defined the rules:

```Make
predict:
 poetry run python -m api.api_requests -e predict

train:
 poetry run python -m api.api_requests -e train
```

However, you can see that there is a lot of **repetition** in the rules. Functions can be used to reduce repetition. The function is defined as follows:

```Make
run-api-command = poetry run python -m api.api_requests -e $(1)

# Make a prediction using the API
predict:
 $(call run-api-command,predict)

# Train the model using the API
train:
 $(call run-api-command,train)
```

In this case, the function is called `run-api-command`. The function takes one argument, which is the argument that is passed to the Python file. The function is called by using the `$(call run-api-command,predict)` syntax. The `$(1)` syntax refers to the first argument that is passed to the function.

As you can see I added some comments in the Makefile, which are especially useful for documenting the Makefile.

### Makefile as a local orchestrator
Makefiles can even be used to locally run **orchestration workflows** that you would run on Airflow in production environments. The following example demonstrates how a Makefile can be used to run an ELT (Extract, Load, Transform) job:

```Make
all: extract load transform 
 @echo "The ELT job is done running."

extract:
 python extract.py

transform:
 python transform.py

load:
 python load.py
```

By running `make all`, the entire ELT workflow can be executed locally.


### Conclusion
In conclusion, Makefiles are a versatile tool that can be used for more than just compilation. They can serve as documentation, run a sequence of tasks, and even locally run complex workflows. By understanding the structure of a Makefile and its rules, developers can create efficient and automated workflows for their projects.

