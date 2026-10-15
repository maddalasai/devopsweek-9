WEEK 9: JENKINS + DOCKER + KUBERNETES

Project image:
141034durga/week9-registration:latest

FILES
- app.py: Flask application routes
- templates/registration.html: registration form
- templates/successful.html: success page
- requirements.txt: Python dependency
- Dockerfile: builds the container image
- Jenkinsfile: build, push, deploy, verify pipeline
- deployment.yaml: Kubernetes Deployment with 4 replicas
- service.yaml: NodePort service on port 30080

BEFORE RUNNING
1. Make sure Docker Desktop and Kubernetes are running.
2. Create a public Docker Hub repository named week9-registration under account 141034durga.
3. In Jenkins, add a Username with password credential:
   ID: dockerhub-creds
   Username: your Docker Hub username
   Password: your Docker Hub access token (recommended)
4. Ensure the Jenkins Windows service account can run docker and kubectl and can access the Kubernetes kubeconfig.
5. Push these files to your own Git repository's main branch.
6. In Jenkins, create a Pipeline job using "Pipeline script from SCM", Git, your repository URL, branch */main, script path Jenkinsfile.
7. Build the job and inspect Console Output.

LOCAL TEST
From this folder:
  docker build -t 141034durga/week9-registration:latest .
  docker run --rm -p 5000:5000 141034durga/week9-registration:latest
Open http://localhost:5000

KUBERNETES ACCESS
After deployment, check:
  kubectl get deployments
  kubectl get pods
  kubectl get services

The Service uses NodePort 30080. If that port is available and Docker Desktop routes it, try:
  http://localhost:30080

If it does not open, use port forwarding:
  kubectl port-forward service/week9-registration-service 18082:5000
Then open http://localhost:18082

SCREENSHOTS TO CAPTURE
1. Project files in VS Code/File Explorer
2. GitHub repository showing the files
3. Docker image in Docker Desktop or output of docker images
4. Docker Hub repository and latest tag
5. Jenkins job configuration
6. Jenkins Console Output showing successful stages
7. kubectl get deployments
8. kubectl get pods (wait for all 4 replicas to be Running)
9. kubectl get services
10. Registration form in browser
11. Successful registration page

Do not put Docker Hub passwords in the Jenkinsfile or GitHub. Use Jenkins credentials.
