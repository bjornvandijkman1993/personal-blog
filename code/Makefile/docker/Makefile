.PHONY: gcp-initial-set-up gcp-login gcp-set-project enable-artifact-registry authenticate-to-registry create-repository build-and-push-image

IMAGE_NAME=hello-world
TAG=latest
PROJECT_ID=makefile-378011
LOCATION=europe-west4
REPOSITORY=hello-world


########### GCP ###########

gcp-initial-set-up: gcp-login gcp-set-project enable-artifact-registry authenticate-to-registry create-repository

gcp-login:
	gcloud auth login

gcp-set-project:
	gcloud config set project $(PROJECT_ID)

enable-artifact-registry:
	gcloud services enable artifactregistry.googleapis.com

authenticate-to-registry:
	gcloud auth configure-docker europe-west4-docker.pkg.dev

create-repository: 
	gcloud artifacts repositories create $(IMAGE_NAME) \
		--repository-format=docker \
		--location=europe-west4 \
		--description="Hello World"

########### Docker ###########

build-and-push-image:
	docker build -t $(IMAGE_NAME):$(TAG) .
	docker tag $(IMAGE_NAME):$(TAG) $(LOCATION)-docker.pkg.dev/$(PROJECT_ID)/$(REPOSITORY)/$(IMAGE_NAME):$(TAG)
	docker push $(LOCATION)-docker.pkg.dev/$(PROJECT_ID)/$(REPOSITORY)/$(IMAGE_NAME):$(TAG)

