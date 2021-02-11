# how-to-install-kubeflow-on-openshift
## Goal
Kubeflow v1.2 on Openshift v4.6.9 (Kubernetes v1.19) via CodeReady Containers on Red Hat Enterprise Linux release 8.3 (Ootpa)

## Options
1. Kubeflow manifest from 
1. kubeflow operator and manifest
1. ODH operator manifest from OpenShift OperatorHub
1. ODH Kubeflow manifests from https://github.com/opendatahub-io/manifests

## Kubeflow Minimum Requirements
4 CPU
50 GB disk
12 GB memory

## CRC Default Configuration
4 vCPUs
9 GB memory
35 GB disk

## CRC Minimal Configuration 
6 vCPUs
16 GB memory
45 GB disk

## CRC Modified Configuration
8 vCPUs
32768 MiB memory
100 GB disk

## Modify CodeReady Containers Default Configuration
1. Create a directory for CRC
1. Download and install CodeReady Containers following these [Installation Instructions](https://access.redhat.com/documentation/en-us/red_hat_codeready_containers/1.21/html/getting_started_guide/introducing-codeready-containers_gsg)
1. Unpack the tar file
1. Move crc into your path
1. Adjust crc config by increasing cpus, memory and disk
```
# Installation
mkdir ~/codeready-containers; cd codeready-containers
wget https://mirror.openshift.com/pub/openshift-v4/clients/crc/latest/crc-linux-amd64.tar.xz
# changes will apply when 'crc start' is run with config complete
crc config set cpus 8
crc config set memory 32768
crc config set disk-size 100
crc config set enable-cluster-monitoring true
crc start
```

## Install Kubeflow
[Kubeflow Instructions](https://www.kubeflow.org/docs/started/k8s/kfctl-k8s-istio/)
1. Create a working directory `working/` (optional)

1. Clone the OpenDataHub manifests from github
1. Create a branch to work from just in case (recommended)

1. Get a copy of the target kfdef file to be installed
1. Modify the manifest for CRC deployment
   1. line 121 remove the comment `#` for crc overlay
   1. line 262 update the value for admin
   1. take note of the metadata namespace value `kubeflow` 

1. Download kfctl v1.2.0 from [Kubeflow releases page](https://github.com/kubeflow/kfctl/releases/tag/v1.2.0)
1. Untar the kfctl file
1. Move the kfctl executable into your $PATH (e.g. /usr/bin) (optional)

1. Create environment variables to simplify deployment (optional)
   1. Set path to base directory to store deployments (e.g. ./base)
   1. Set the Kubeflow application directory for this deploymeny (e.g. ./base/kubeflow)
   1. Create the BASE_DIR directory and change into it

1. login to your cluster from the cli using `oc login`
   1. you can retrieve your credentials running `crc console --credentials` from the cli
1. Create the namespace from the manifest `kubeflow`
1. Build the deployment configuration
1. Verify a new `kustomize` folder has been created with a series of yaml files
1. Apply the kfdef
1. Verify all pods are `running`
   1. Monitor progress from the web ui under Project Kubeflow from Workloads > Pods
1. Get the ingress route to the Kubeflow Dashboard

```1
mkdir working; cd working

git clone https://github.com/opendatahub-io/manifests.git
git checkout -b mybranch

wget https://raw.githubusercontent.com/opendatahub-io/manifests/v1.0-branch-openshift/kfdef/kfctl_openshift.yaml
vim kfctl_openshift.yaml

wget https://github.com/kubeflow/kfctl/releases/download/v1.2.0/kfctl_v1.2.0-0-gbc038f9_linux.tar.gz
tar -xvf kfctl_v1.2.0-0-gbc038f9_linux.tar.gz 
mv ./kfctl /usr/bin

export BASE_DIR=./base
mkdir $BASE_DIR; cd $BASE_DIR
export KF_DIR=${KF_BASE}/${KF_NAME}

crc console --credentials
oc login -u kubeadmin -p <the-password> https://api.crc.testing:6443
oc new-project kubeflow
kfctl build -f kfctl_openshift.yaml
tree ./kustomize
kfctl apply -f kfctl_openshift.yaml
oc get pods -n kubeflow -wi
oc get routes -n istio-system istio-ingressgateway -o jsonpath='http://{.spec.host}/'
```
