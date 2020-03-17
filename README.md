## IBM APP Connect Enteprise Deployment using Tekton Pipelines

Tekton is an open source project to configure and run CI/CD pipelines within a Kubernetes cluster.


## Introduction

Example file set to build and deploy IBM ACE

## Instructions

For use in conjunction with source Git repos - https://github.com/DAVEXACOM/ibm-ace-mqc-soe-ms2-build

src\Dockerfile in this repos is copy of the docker file in https://github.com/DAVEXACOM/ibm-ace-mqc-soe-ms2-build

acemsdep\ibmace.yaml creates the deployment config in RHOS and has a copy in the https://github.com/DAVEXACOM/ibm-ace-mqc-soe-ms2-build
acemsdep\create-tt-acems2-service.yaml - a service definition to expose the ace deployment created by tekton - I have not included this in the pipeline - I manually create it in RHOS
acemsdep\create-tt-acems2-route.yaml - a route definition to expose the ace deployment created by tekton - I have not included this in the pipeline - I manually create it in RHOS
tekton directory has the tekton YAML files for building and deploying with
	a) Kaniko - do not use these - kaniko has a restriction that it must run as root. ACE builds don't allow that. The pipeline works but the image crashloops. Wait until Kaniko remove the restriction.
	b) Buildah - I took the published standard Buildah clustertask and refactored it as a ("user") task with parameter and resource settings that are a match for Kaniko. This way the rest of the tekton artifacts remain unchanged and Kaniko and buildah source-to-image( build and push) tasks can be swapped in and out when Kaniko has the root user restriction removed at some point.

Use OC CREATE commands to load the tekton artifacts into your RHOS environment
resources.yaml
source-to-image-buildah.yaml
deploy-using-kubectl-common.yaml
build-and-deploy-pipeline-buildah.yaml
acems2-pipeline-buildah-run.yaml

oc create -f c:\users\DAVIDARNOLD\IBM\gitREPOS\tekton-ace-example\tekton\run\acems2-pipeline-buildah-run.yaml - this is the last command - it creates the "run" which starts the pipeline

USE OC REPLACE if you change and files
oc replace -f c:\users\DAVIDARNOLD\IBM\gitREPOS\tekton-ace-example\tekton\tasks\source-to-image-buildah.yaml

USE OC DELETE -f c:\temp\tempyaml.yaml to remove any artifacts from RHOS - use a yaml snippet in a file to identify kind: , name: and namespace: see example tempyaml.yaml below

apiVersion: tekton.dev/v1alpha1
kind: Task
metadata:
  creationTimestamp: '2020-03-03T00:27:03Z'
  generation: 3
  name: source-to-image
  namespace: da-build-project
## Test ACE MS2 
POST
http://tt-ibm-ace-mqc-soe-ms2-build-da-build-project.apps.cloudpak.ocp4.cloudnativekube.com/microservice2/v1/message
with data
{"Messages":["Hello From Test Client"]}
should return
{"Messages":["Hello From Test Client","Hello From MicroService 2 after change tekton  test"]}