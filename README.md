# k8s-project

This project demonstrates the integration between Kubernetes, Jenkins, DockerHub, and GitHub. The goal is to automate the deployment of a simple Flask application using a CI/CD pipeline. When changes are pushed to the `app.py` file on GitHub, Jenkins builds a Docker image, pushes it to DockerHub, and deploys it to a Kubernetes cluster.

## Project Description
In this project, I integrated Kubernetes, Jenkins, DockerHub, and GitHub. The goal is that whenever I modify something in the app.py file (a simple Flask application that generates an HTML page displaying "hello world Yahav"), and push it to GitHub, a predefined trigger activates Jenkins. The Jenkins pipeline, defined in the Jenkinsfile stored in GitHub, retrieves the app.py file, builds it into a Docker image, and push it to DockerHub. Each time, it automatically increments the image version number by 1. Moreover, Jenkins deploys the image to Kubernetes, initiating a deployment process to ensure immediate availability of the application.

## Plugins Used
- **Kubernetes CLI Plugin**
- **Kubernetes Continuous Deploy Plugin**
- **Kubernetes :: Pipeline :: DevOps Steps**
- **Kubernetes Credentials Plugin**
- **Kubernetes Plugin**
- **Docker Plugin**
- **Docker Pipeline**
- **Docker Commons Plugin**
- **Docker API Plugin**
- **docker-build-step**
- **Blue Ocean**
- **Git Plugin**
- **GitHub Pipeline for Blue Ocean**
- **GitHub Plugin**

## Project Workflow

![287505556-5653c1fa-d537-4f41-9f87-48e130fb0b72](https://github.com/yahav123456/k8s_project/assets/166650066/dfa0d000-b8b9-49bb-b2ab-17c565f8d8ef)

1. **Setup Kubernetes Cluster**
    - Created a Kubernetes cluster using Kind and configured it with `jenkins-config.yaml`.
    - Configured namespace, service account, role binding, and token using `jenkins-token.yaml`.

    ```sh
    kind create cluster --config jenkins-config.yaml
    kubectl cluster-info --context kind-kind
    kubectl create namespace jenkins
    kubectl create serviceaccount jenkins --namespace=jenkins
    kubectl -n jenkins create -f jenkins-token.yaml
    kubectl -n jenkins describe secret jenkins-token
    kubectl create rolebinding jenkins-admin-binding --clusterrole=admin --serviceaccount=jenkins:jenkins --namespace=jenkins
    ```

2. **Run Jenkins Container**
    - Started Jenkins container.

    ```sh
    docker run -d -p 8888:8080 -p 50000:50000 --name jenkins jenkins/jenkins:lts
    ```

    ![צילום מסך 2024-05-26 184147](https://github.com/yahav123456/k8s_project/assets/166650066/9fef6449-f04d-4aa2-9abe-b3a5e102584b)


3. **Integrate Jenkins with Kubernetes**
    - Connected Jenkins to Kubernetes by configuring the Kubernetes cloud settings and using the token from the previous step.

   ![צילום מסך 2024-05-27 100349](https://github.com/yahav123456/k8s_project/assets/166650066/47aa4b48-3f78-4125-a9e1-e2a9331c9878)


4. **Connect Jenkins to GitHub**
    - Configured a GitHub webhook and Jenkins credentials to trigger the pipeline on code changes.
    - Used Ngrok to expose Jenkins to the public for GitHub webhook integration.
    - **Note that each time Ngrok is restarted, the URL changes, requiring updating the webhook in GitHub accordingly.**


   ![צילום מסך 2024-05-26 184119](https://github.com/yahav123456/k8s_project/assets/166650066/839869af-ca06-41fa-8ad5-4c4bd51abd5b)
  ![צילום מסך 2024-05-27 102349](https://github.com/yahav123456/k8s_project/assets/166650066/027dbddf-224e-4da4-a1a3-31e3575da0ca)


5. **Connect Jenkins to DockerHub**
    - Used Jenkins credentials and DockerHub access tokens for image pushing.
  
    

6. **Configure Jenkins Pipeline**
    - Set up the pipeline to trigger on changes to `app.py` and defined the steps in the `Jenkinsfile`.

   ![צילום מסך 2024-05-26 182507](https://github.com/yahav123456/k8s_project/assets/166650066/26eb8e99-6c8d-4771-ae16-a1dc9f4bb783)

7. **Run the Pipeline**
    - Initial pipeline run showed Docker commands were not found.
    - Resolved by installing Docker inside the Jenkins container and starting the Docker daemon.

    ```sh
    apt-get update
    apt-get install sudo -y
    sudo usermod -aG docker jenkins
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

    - Configured kubeconfig in a `config.yaml` file in GitHub.

  ![צילום מסך 2024-05-27 122231](https://github.com/yahav123456/k8s_project/assets/166650066/3d7bb440-f8b3-4b48-92d9-ace136eec102)
   ![צילום מסך 2024-05-26 183637](https://github.com/yahav123456/k8s_project/assets/166650066/59da61ec-b25e-4ae5-9b60-e177c2709ce1)


9. **Deployment Verification**
    - Verified the deployment using Lens, ensuring new pods replace the old ones.

  ![צילום מסך 2024-05-26 182005](https://github.com/yahav123456/k8s_project/assets/166650066/e717f453-40c1-4d61-bb3a-3c8cf794c076)

   ![צילום מסך 2024-05-26 183358](https://github.com/yahav123456/k8s_project/assets/166650066/439dae3c-3d8f-484a-b1be-0890d9f05cc8)


## Troubleshooting

### Docker Commands Not Found
- **Issue**: Jenkins container did not recognize Docker commands.
- **Solution**: Installed Docker Inside the Jenkins container and started the Docker daemon manually.

### kubectl Commands Not Found
- **Issue**: Jenkins container did not recognize `kubectl` commands.
- **Solution**: Installed kubectl manually and configured kubeconfig.

### GitHub Webhook and Localhost Issues
- **Issue**: Webhook did not work due to Jenkins running on localhost.
- **Solution**: Used Ngrok to expose Jenkins to the public and allow GitHub to trigger the webhook.

## Conclusion
The integration of Kubernetes, Jenkins, DockerHub, and GitHub successfully automated the deployment of the Flask application. The CI/CD pipeline ensures that any change in the application code triggers the build, image creation, push to DockerHub, and deployment to Kubernetes automatically.
