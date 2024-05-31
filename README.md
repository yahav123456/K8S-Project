# k8s-project

This project showcases how Kubernetes, Jenkins, DockerHub, ArgoCD, and GitHub work together. Its aim is to automate the deployment process of a basic Flask application using CI/CD. Whenever modifications are made to the app.py file on GitHub, Jenkins build a Docker image, uploads it to DockerHub, and then modifies the deployment.yaml file. This triggers ArgoCD to deploy the updated application to the Kubernetes cluster.

## Plugins Used
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
    kubectl config get-contexts (to verify the cluster)
    ```

2. **Run Jenkins Container**
    - Build a Docker image containing Jenkins with Docker installed inside using a Dockerfile.
    - Started Jenkins container.

    ```sh
     docker run -p 9091:8080 -p 50000:50000 --name jenkins -v jenkins_home:/var/jenkins_home myappjenkins
    ```

    ![צילום מסך 2024-05-26 184147](https://github.com/yahav123456/k8s_project/assets/166650066/9fef6449-f04d-4aa2-9abe-b3a5e102584b)


3. **Configure ArgoCD**
   - This will create a new namespace, ArgoCD, where ArgoCD services and application resources will live.
     ```sh
     kubectl create namespace argocd
     kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
     ```
   - How to get the url of the ArgoCD?
     ```sh
     kubectl get svc -n argocd
     kubectl port-forward svc/argocd-server -n argocd 9090:443
     ```
    ![צילום מסך 2024-05-31 022043](https://github.com/yahav123456/k8s_project/assets/166650066/3bbd540b-934b-474f-9cf9-f5f16052dc74)
   - ArgoCD Username & Password.
     ```sh
     ArgoCD username : admin
     ArgoCD password : argocd admin initial-password -n argocd
     ```
   - To synchronize ArgoCD with the cluster, apply the `application.yaml` file.
     ```sh
     kubectl apply -f application.yaml
     ```
     
     ![צילום מסך 2024-05-31 013822](https://github.com/yahav123456/k8s_project/assets/166650066/881b5162-1a7c-4b47-8540-81a5496964b1)

4. **Connect Jenkins to GitHub**
    - Set up a GitHub webhook and configured Jenkins credentials to trigger the pipeline upon a commit.
    - Used Ngrok to expose Jenkins to the public for GitHub webhook integration.
    - **Note that each time Ngrok is restarted, the URL changes, requiring updating the webhook in GitHub accordingly.**


   ![צילום מסך 2024-05-26 184119](https://github.com/yahav123456/k8s_project/assets/166650066/839869af-ca06-41fa-8ad5-4c4bd51abd5b)
  ![צילום מסך 2024-05-27 102349](https://github.com/yahav123456/k8s_project/assets/166650066/027dbddf-224e-4da4-a1a3-31e3575da0ca)


5. **Connect Jenkins to DockerHub**
    - Used Jenkins credentials and DockerHub access tokens for image pushing.
      

6. **Configure Jenkins Pipeline [jenkinsfile]**
    - **Checkout:**
        - **Purpose:** This stage ensures that the latest code from the specified branch in the GitHub repository is checked out for further processing.
        - **Steps:**
            - It retrieves the author's name of the last commit made to the repository.
            - If the author is identified as "Jenkins", it skips the build to avoid infinite loops where Jenkins continuously builds itself.
            - It then proceeds to checkout the code from the specified branch in the GitHub repository using the provided credentials.
    - **Build Docker Image:**
        - **Purpose:** This stage builds a Docker image of the Flask application using the Dockerfile present in the repository.
        - **Steps:**
            - It utilizes Docker's build functionality to construct the Docker image with the specified tag.
    - **Push to DockerHub:**
        - **Purpose:** This stage pushes the built Docker image to DockerHub for storage and distribution.
        - **Steps:**
            - It uses Docker's registry functionality along with DockerHub credentials to authenticate and push the Docker image.
    - **Update Kubernetes Deployment.yaml:**
        - **Purpose:** This stage updates the Kubernetes deployment manifest file (deployment.yaml) with the latest version of the Docker image.
        - **Steps:**
            - It employs a sed command to replace the existing image tag in the deployment.yaml file with the newly built Docker image's tag.
            - Then, it commits this change to the GitHub repository, using Jenkins' credentials to authenticate, along with the provided username and password.
            - Finally, it pushes the updated deployment.yaml file to the GitHub repository's main branch.

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
   

 - And now the pipeline works flawlessly
 ![צילום מסך 2024-05-31 121354](https://github.com/yahav123456/k8s_project/assets/166650066/4c38928a-0d17-437e-94f3-49fa72d2b6da)
![צילום מסך 2024-05-31 121431](https://github.com/yahav123456/k8s_project/assets/166650066/4a1b0625-7654-4c89-888c-f00093fc88dd)

      
10. **Deployment Verification**
    - Verified the deployment using the ArgoCD interface, ensuring new pods replace the old ones.
    - So ,How can I access the Flask application after the deployment?
    ```sh
    kubectl port-forward pod/<pod name> 8080:80 -n <name space>
    ```
![צילום מסך 2024-05-31 021342](https://github.com/yahav123456/k8s_project/assets/166650066/3fce5f8c-64be-4195-a083-e71d3998eb7f)
 ![צילום מסך 2024-05-31 122048](https://github.com/yahav123456/k8s_project/assets/166650066/3bb94e5a-07d4-4b2f-8edf-5a4157c32b8c)

## Troubleshooting

### Docker Commands Not Found
- **Issue**: Jenkins container did not recognize Docker commands.
- **Solution**: starting the Docker daemon manually.

### Infinite Loop in Pipeline
- **Issue**: Jenkins entered an infinite loop, as every time it pushed changes to the deployment.yaml file, it triggered itself, causing a continuous loop.
- **Solution**: Added a condition in the pipeline to skip the build if the author of the last commit is identified as "Jenkins", preventing Jenkins from continuously building itself.

### GitHub Webhook and Localhost Issues
- **Issue**: Webhook did not work due to Jenkins running on localhost.
- **Solution**: Used Ngrok to expose Jenkins to the public and allow GitHub to trigger the webhook.

## Summary
The pipeline workflow begins by checking out the latest code from a GitHub repository. It then builds a Docker image of a Flask application, pushes it to DockerHub, and updates the Kubernetes deployment manifest file with the new image version. Finally, it commits and pushes this change back to the GitHub repository. The outcome of the pipeline is an automated deployment of the Flask application on a Kubernetes cluster. Whenever changes are made to the app.py file and pushed to GitHub, Jenkins automatically triggers the pipeline, ensuring that the latest version of the application is built, deployed, and updated in the Kubernetes cluster without manual intervention.

Following the pipeline's completion, ArgoCD takes over, detecting changes in the Git repository's Kubernetes manifests. It then synchronizes the live state of the application with the desired state specified in the Git repository. This ensures that any updates pushed through the pipeline are automatically deployed to the Kubernetes cluster, maintaining consistency between the defined configuration in Git and the actual running application.
