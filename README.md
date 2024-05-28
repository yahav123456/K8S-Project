# k8s-project

This project demonstrates the integration between Kubernetes, Jenkins, DockerHub, and GitHub. The goal is to automate the deployment of a simple Flask application using a CI/CD pipeline. When changes are pushed to the `app.py` file on GitHub, Jenkins builds a Docker image, pushes it to DockerHub, and deploys it to a Kubernetes cluster.

## Plugins Used
- **Kubernetes CLI Plugin**
- **Kubernetes Continuous Deploy Plugin**
- **Kubernetes Credentials Plugin**
- **Kubernetes Plugin**
- **Docker Plugin**
- **Docker Commons Plugin**
- **Docker-build-step**
- **Git Plugin**
- **GitHub Plugin**
- **Blue Ocean**

## Project Workflow

![1414k8s_](https://github.com/yahav123456/k8s_project/assets/166650066/36e23b15-1957-4698-888e-e6a5841e1713)


1. **Setup Kubernetes Cluster**
    - Created a Kubernetes cluster using Kind and configured it with `jenkins-config.yaml`.

    ```sh
    kind create cluster --config jenkins-config.yaml
    kubectl cluster-info --context kind-kind
    kubectl create namespace jenkins
    kubectl config set-context --current --namespace=jenkins (to connect the namespace to the cluster)
    ```

2. **Run Jenkins Container**
    - Build a Docker image containing Jenkins with Docker installed inside using a Dockerfile.
    - Started Jenkins container.

    ```sh
     docker run -p 9091:8080 -p 50000:50000 --name jenkins -v jenkins_home:/var/jenkins_home myappjenkins
    ```

    ![צילום מסך 2024-05-26 184147](https://github.com/yahav123456/k8s_project/assets/166650066/9fef6449-f04d-4aa2-9abe-b3a5e102584b)


3. **Integrate Jenkins with Kubernetes**
    - Connected Jenkins to Kubernetes by configuring the Kubernetes cloud settings and using the kubeconfig credentials defined within Jenkins.
    - **The kubeconfig file details are located in the .kube directory at the following path: C:/Users/Username/.kube**

   ![צילום מסך 2024-05-27 100349](https://github.com/yahav123456/k8s_project/assets/166650066/47aa4b48-3f78-4125-a9e1-e2a9331c9878)


4. **Connect Jenkins to GitHub**
    - Set up a GitHub webhook and configured Jenkins credentials to trigger the pipeline upon a commit.
    - Used Ngrok to expose Jenkins to the public for GitHub webhook integration.
    - **Note that each time Ngrok is restarted, the URL changes, requiring updating the webhook in GitHub accordingly.**


   ![צילום מסך 2024-05-26 184119](https://github.com/yahav123456/k8s_project/assets/166650066/839869af-ca06-41fa-8ad5-4c4bd51abd5b)
  ![צילום מסך 2024-05-27 102349](https://github.com/yahav123456/k8s_project/assets/166650066/027dbddf-224e-4da4-a1a3-31e3575da0ca)


5. **Connect Jenkins to DockerHub**
    - Used Jenkins credentials and DockerHub access tokens for image pushing.
      

6. **Configure Jenkins Pipeline**
   - agent any: This specifies that the pipeline can run on any available Jenkins agent.
     environment: Defines environment variables for Docker Hub credentials, Git credentials, Docker image name, and the build version.

  
  - stage('Checkout'): Checks out the code from the main branch of the specified GitHub repository using the provided Git credentials.

  
  - stage('Build Docker Image'): Builds a Docker image from the checked-out code, tagging it with the version number based on the Jenkins build number.

  
  - stage('Push to DockerHub'): Authenticates with Docker Hub using the provided credentials and pushes the built Docker image to the Docker Hub repository.

  
  - stage('Deploy to Kubernetes'): Deploys the new Docker image to a Kubernetes cluster.
    environment: Sets the Kubernetes configuration using the provided credentials.
    script: Defines deployment details and updates the Kubernetes deployment with the new Docker image version, recording the change.

   ![צילום מסך 2024-05-26 182507](https://github.com/yahav123456/k8s_project/assets/166650066/26eb8e99-6c8d-4771-ae16-a1dc9f4bb783)
    

7. **Run the Pipeline**
    - Initial pipeline run showed Docker commands were not found.
    - Resolved by starting the Docker daemon inside the Jenkins container.

    ```sh
    apt-get update
    apt-get install sudo -y
    chown root:docker /var/run/docker.sock
    sudo dockerd --host=unix:///var/run/docker.sock --host=tcp://0.0.0.0:2375 &
    ```

   ![צילום מסך 2024-05-27 124425](https://github.com/yahav123456/k8s_project/assets/166650066/6ed3c963-67b3-4e5b-bd59-9319f7c2177e)

 - After this, Jenkins recognized Docker commands, and the pipeline continued to work. The image was successfully created and pushed to DockerHub.

   ![צילום מסך 2024-05-26 182610](https://github.com/yahav123456/k8s_project/assets/166650066/f445a69e-a41d-4b92-be02-b46ef6d30dc4)
   

8. **Resolve Deployment Issues**
    - Encountered `kubectl: not found` error, resolved by installing kubectl inside the Jenkins container.

    ```sh
    apt-get update && apt-get install -y apt-transport-https gnupg2 curl
    curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
    chmod +x ./kubectl
    ```

    - Configured kubeconfig using the kubeconfig credentials from Jenkins.

  ![צילום מסך 2024-05-27 122231](https://github.com/yahav123456/k8s_project/assets/166650066/3d7bb440-f8b3-4b48-92d9-ace136eec102)
   ![צילום מסך 2024-05-26 183637](https://github.com/yahav123456/k8s_project/assets/166650066/59da61ec-b25e-4ae5-9b60-e177c2709ce1)

   - I utilize a simple deployment.yaml file to deploy a random application. This file contains configurations defining the container's name, deployment name, and 
     namespace. After the pipeline execution, the existing deployment is deleted, and a new application is created using the Docker image from our DockerHub repository, 
     ensuring a fresh deployment each time.
     
9. **Deployment Verification**
    - Verified the deployment using Lens, ensuring new pods replace the old ones.

  ![צילום מסך 2024-05-26 182005](https://github.com/yahav123456/k8s_project/assets/166650066/e717f453-40c1-4d61-bb3a-3c8cf794c076)

   ![צילום מסך 2024-05-26 183358](https://github.com/yahav123456/k8s_project/assets/166650066/439dae3c-3d8f-484a-b1be-0890d9f05cc8)


## Troubleshooting

### Docker Commands Not Found
- **Issue**: Jenkins container did not recognize Docker commands.
- **Solution**: starting the Docker daemon manually.

### kubectl Commands Not Found
- **Issue**: Jenkins container did not recognize `kubectl` commands.
- **Solution**: Installed kubectl manually and configured kubeconfig.

### GitHub Webhook and Localhost Issues
- **Issue**: Webhook did not work due to Jenkins running on localhost.
- **Solution**: Used Ngrok to expose Jenkins to the public and allow GitHub to trigger the webhook.

## Summary
The integration of Kubernetes, Jenkins, DockerHub, and GitHub successfully automated the deployment of the Flask application. The CI/CD pipeline ensures that any change in the application code triggers the build, image creation, push to DockerHub, and deployment to Kubernetes automatically.
