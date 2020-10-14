# WIP - Testing on OKD 4.5

The files for this installation are modified from: https://github.com/openshift/tektoncd-pipeline-operator

Clone this repo:

```bash
git clone https://github.com/cgruver/tekton-pipeline-okd4.git
```

### Mirror Images to your local registry:

The next step is to pull all of the images needed for the install, and make them available to the OpenShift cluster.

These instructions assume that you are using a personal workstation with Docker desktop.  You can also use `podman`, `skopeo`, or `buildah` to perform these steps.

The list of images is in the file `tkn-images`, included with this guide.

From your internet connected workstation or bastion host:

1. Pull the images needed for the install:

    Replace the value for LOCAL_REGISTRY with the URL for your registry.

    ```bash
    export LOCAL_REGISTRY=nexus.your.domain.org:5000

    cd tekton-pipeline-okd4

    for i in $(cat ./disconnected-install/tkn-images)
    do 
        IMAGE_TAG=${LOCAL_REGISTRY}/$(echo $i | cut -d"/" -f2-)
        docker pull ${i}
        docker tag ${i} ${IMAGE_TAG}
    done
    ```

1. Log into your local image registry.  Since you are in a disconnected environment, you might have to change networks for this step.

    ```bash
    docker login ${LOCAL_REGISTRY}
    ```

1. Push the newly tagged images.

    ```bash
    for i in $(cat ./disconnected-install/tkn-images)
    do 
        IMAGE_TAG=${LOCAL_REGISTRY}/$(echo $i | cut -d"/" -f2-)
        docker push ${IMAGE_TAG}
    done
    ```

### Install Tekton

1. Prepare a working directory:

    ```bash
    mkdir -p ~/tkn-workdir
    cp -r ./disconnected-install/addons ~/tkn-workdir
    cp ./disconnected-install/*.yaml ~/tkn-workdir
    ```

1. Prepare the installation files:

    ```bash
    cd ~/tkn-workdir
    for i in $(find . | grep yaml)
    do
        sed -i "s|--LOCAL_REGISTRY--|${LOCAL_REGISTRY}|g" ${i}
    done
    ```

1. Install Tekton: (Log into your OKD cluster as a `cluster-admin`)

    ```bash
    oc apply -f 00-release.yaml	
    oc apply -f 01-clusterrole.yaml
    oc apply -f 02-rolebinding.yaml
    oc apply -f tektoncd-triggers-v0.8.1.yaml

    for i in $(find addons | grep yaml)
    do
        oc apply -f ${i}
    done
    ```
