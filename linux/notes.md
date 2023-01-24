 # Note - to do before start.sh

Tasks might depend on executables that your container is expected to provide. For instance, you must add the zip and unzip packages to the RUN apt-get command in order to run the ArchiveFiles and ExtractFiles tasks. Also, as this is a Linux Ubuntu image for the agent to use, you can customize the image as you need. E.g.: if you need to build .NET applications you can follow the document Install the .NET SDK or the .NET Runtime on Ubuntu and add that to your image.

# Note - before 1-docker-build.sh

You must also use a container orchestration system, like Kubernetes or Azure Container Instances, to start new copies of the container when the work completes.

#  Note - before 2-docker-run-*.sh

Optionally, you can control the pool and agent work directory by using additional environment variables.

## Environment variables
Environment variable    Description
AZP_URL                 The URL of the Azure DevOps or Azure DevOps Server instance.
AZP_TOKEN               Personal Access Token (PAT) with Agent Pools (read, manage) scope, created by a user who has permission to configure agents, at AZP_URL.
AZP_AGENT_NAME          Agent name (default value: the container hostname).
AZP_POOL	            Agent pool name (default value: Default).
AZP_WORK                Work directory (default value: _work).

## Use Docker within a Docker container
In order to use Docker from within a Docker container, you bind-mount the Docker socket.

### **Caution**

Doing this has serious security implications. The code inside the container can now run as root on your Docker host.

If you're sure you want to do this, see the bind mount documentation on Docker.com.

## Use Azure Kubernetes Service cluster

### **Caution**

Please, consider that any docker based tasks will not work on AKS 1.19 or earlier due to docker in docker restriction. Docker was replaced with containerd in Kubernetes 1.19, and Docker-in-Docker became unavailable.

## Deploy and configure Azure Kubernetes Service
https://learn.microsoft.com/en-us/azure/aks/kubernetes-walkthrough-portal

## Deploy and configure Azure Container Registry
https://learn.microsoft.com/en-us/azure/container-registry/container-registry-get-started-portal

## Configure secrets and deploy a replica set
1. Create the secrets on the AKS cluster:
kubectl create secret generic azdevops \
  --from-literal=AZP_URL=https://dev.azure.com/yourOrg \
  --from-literal=AZP_TOKEN=YourPAT \
  --from-literal=AZP_POOL=NameOfYourPool

2. Run this command to push your container to Container Registry:
docker push <acr-server>/dockeragent:latest

3. Configure Container Registry integration for existing AKS clusters
_If you have multiple subscriptions on the Azure Portal, please, use this command first to select a subscription_

az account set --subscription <subscription id> or <subscription name>

az aks update -n myAKSCluster -g myResourceGroup --attach-acr <acr-name>

4. ~/AKS/ReplicationController.yaml

5. run
kubectl apply -f ReplicationController.yaml

## Set custom MTU parameter
Allow specifying MTU value for networks used by container jobs (useful for docker-in-docker scenarios in k8s cluster).

You need to set the environment variable AGENT_MTU_VALUE to set the MTU value, after that restart the self-hosted agent. 

1. agent restart: https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/v2-windows?view=azure-devops#how-do-i-restart-the-agent

2. environment variables for each individual agent: https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/v2-windows?view=azure-devops#how-do-i-set-different-environment-variables-for-each-individual-agent

## Mounting volumes using Docker within a Docker container
If a Docker container runs inside another Docker container, they both use host's daemon, so all mount paths reference the host, not the container.

1. For example, if we want to mount path from host into outer Docker container, we can use this command:
docker run ... -v <path-on-host>:<path-on-outer-container> ...

2. And if we want to mount path from host into inner Docker container, we can use this command:
docker run ... -v <path-on-host>:<path-on-inner-container> ...

3. But we can't mount paths from outer container into the inner one; to work around that, we have to declare an ENV variable:
docker run ... --env DIND_USER_HOME=$HOME ...

4. After this, we can start the inner container from the outer one using this command:
docker run ... -v $DIND_USER_HOME:<path-on-inner-container> ..

# Common Errors
If you're using Windows, and you get the following error:

shell
‘standard_init_linux.go:178: exec user process caused "no such file or directory"

1. Install Git Bash by downloading and installing git-scm.

2. Run this command:

shell
dos2unix ~/dockeragent/Dockerfile
dos2unix ~/dockeragent/start.sh
git add .
git commit -m 'Fixed CR'
git push

3. Try again. You no longer get the error.


# Articles
Self-hosted Windows agents https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/v2-windows?view=azure-devops
Self-hosted Linux agents https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/v2-linux?view=azure-devops
Self-hosted macOS agents https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/v2-osx?view=azure-devops
Microsoft-hosted agents https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/hosted?view=azure-devops