# Jenkins GitOps Docker

This project is the first part of a CI/CD pipeline based on the guide ["Building a Robust CI/CD Pipeline with Jenkins, Docker, Kubernetes, and ArgoCD"](https://sameeradissanayaka.medium.com/building-a-robust-ci-cd-pipeline-with-jenkins-docker-kubernetes-and-argocd-bdcc15a31a2f) by Sameera Dissanayaka. However, this implementation has been adapted for macOS instead of Linux.

## Differences from the Original Guide

While the original guide provides a comprehensive walkthrough, there are some key differences and additional steps required for macOS:

1. **Jenkins Installation**:
   - Jenkins was installed using `brew` on macOS. This method simplifies installation but introduces communication issues between Jenkins and Docker due to macOS's sandboxing and permissions model.
   - To resolve these issues, the following command was used to ensure Jenkins and Docker can communicate properly:
     ```bash
     launchctl setenv PATH /opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin
     ```
   - Jenkins on macOS is managed using the following commands:
     - Start Jenkins: `brew services start jenkins`
     - Stop Jenkins: `brew services stop jenkins`
     - Restart Jenkins: `brew services restart jenkins`

2. **Docker Read-Write Authentication**:
   - The original guide does not mention the creation of a read-write authentication for Docker. This step is necessary to push Docker images to Docker Hub.

3. **Required Jenkins Plugins**:
   - Install the following plugins in Jenkins for the pipeline to work correctly:
     - **Docker** plugin
     - **Docker Pipeline** plugin

4. **Pipeline Naming**:
   - The second pipeline (used for updating the Kubernetes manifest) must be named the same as the job triggered in the last step of the `Jenkinsfile`:
     ```groovy
     build job: 'updatemanifest', parameters: [string(name: 'DOCKERTAG', value: env.BUILD_NUMBER)]
     ```
     In this case, the pipeline should be named `updatemanifest`.

5. **Push Image Stage**:
   - The `Push Image` stage in the pipeline was modified to address issues encountered during execution. Ensure the updated commands are used to push Docker images successfully.

6. **Credentials Configuration**:
   - Ensure that the Docker Hub credentials ID in Jenkins matches the one specified in the `Jenkinsfile`:
     ```groovy
     withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')])
     ```

## Project Overview

This project builds a Docker image for a simple Flask application and pushes it to Docker Hub. The pipeline includes the following stages:

1. **Clone Repository**: Pulls the source code from the Git repository.
2. **Build Image**: Builds a Docker image for the Flask application.
3. **Test Image**: Runs basic tests inside the Docker container.
4. **Push Image**: Pushes the Docker image to Docker Hub.
5. **Trigger Manifest Update**: Triggers the second pipeline (`updatemanifest`) to update the Kubernetes deployment manifest with the new Docker image tag.

## Notes

- Follow the original guide for the overall setup, but refer to this README for macOS-specific adjustments.
- Ensure all credentials and pipeline configurations are correctly set up in Jenkins to avoid errors during execution.

By following these steps, you can successfully implement the first part of the CI/CD pipeline on macOS.

Working jenkins should look like this:
![jenkins](./img/Screenshot%202025-05-12%20at%204.55.47 PM.png)

Note that the first pipeline can be named whatever you want, but the second pipeline should be named `updatemanifest` as mentioned above. The first pipeline is used to build the docker image and push it to docker hub, while the second pipeline is used to update the kubernetes deployment manifest with the new docker image tag.

The second pipeline and it's code are found [here](https://github.com/FelipeBarretoB/jenkins-gitops-k8s)