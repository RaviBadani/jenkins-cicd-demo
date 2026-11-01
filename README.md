# Experiment 13 – Jenkins CI/CD with Docker

This project demonstrates a local CI/CD pipeline for a Python Flask application using Jenkins, Docker, Git, and GitHub.

## Project files

- `app.py` – Flask sample application
- `requirements.txt` – Python dependencies
- `test_app.py` – automated tests using pytest
- `Dockerfile` – builds the application image
- `Dockerfile.jenkins` – builds Jenkins with Docker CLI, Python, pip, and Git
- `Jenkinsfile` – checkout, install, test, build, deploy, and verify stages

## Prerequisites (Windows)

Install Docker Desktop and Git. Start Docker Desktop using the Linux container engine.

Check in PowerShell:

```powershell
git --version
docker --version
docker run hello-world
```

## A. Test the application with Docker

Open PowerShell in this project folder:

```powershell
docker build -t jenkins-cicd-demo .
docker run -d -p 5000:5000 --name demo-app jenkins-cicd-demo
docker ps
```

Open http://localhost:5000 and http://localhost:5000/health.

Stop and remove this test container before Jenkins uses the same name:

```powershell
docker stop demo-app
docker rm demo-app
```

## B. Build and start Jenkins

```powershell
docker build -t my-jenkins -f Dockerfile.jenkins .
docker volume create jenkins_home
```

Start Jenkins (PowerShell line continuation uses the backtick):

```powershell
docker run -d --name jenkins `
  -p 8080:8080 `
  -p 50000:50000 `
  -v jenkins_home:/var/jenkins_home `
  -v /var/run/docker.sock:/var/run/docker.sock `
  my-jenkins
```

The Docker socket mount is intended for Docker Desktop's Linux container engine. It gives Jenkins control over the host Docker daemon. Use this only on a trusted local lab machine.

Unlock Jenkins:

```powershell
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

Visit http://localhost:8080, enter the password, install suggested plugins, and create an administrator account.

Confirm these plugins are installed under **Manage Jenkins → Plugins**:
- Pipeline
- Git
- GitHub (optional for webhook triggering)

## C. Push the project to GitHub

Create an empty GitHub repository named `jenkins-cicd-demo`. In this project folder, run:

```powershell
git init
git add .
git commit -m "Initial CI/CD project"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/jenkins-cicd-demo.git
git push -u origin main
```

Replace `YOUR_USERNAME` with your GitHub username.

## D. Create the Jenkins pipeline job

1. In Jenkins, choose **New Item**.
2. Name it `CI-CD-Pipeline` and select **Pipeline**.
3. Under **Pipeline**, choose **Pipeline script from SCM**.
4. SCM: **Git**.
5. Repository URL: `https://github.com/YOUR_USERNAME/jenkins-cicd-demo.git`.
6. Branch specifier: `*/main`.
7. Script path: `Jenkinsfile`.
8. Save, then choose **Build Now**.

The Jenkinsfile uses `pollSCM` to check for commits approximately every two minutes. Jenkins must be able to access the repository. For a private repository, configure Git credentials in Jenkins and select them in the job's SCM configuration.

## E. Verify

A successful build should show:
1. Checkout
2. Install Dependencies
3. Run Tests
4. Build Docker Image
5. Deploy Container Locally
6. Verify Deployment

Open http://localhost:5000 to verify the deployed application.

## F. Test automatic build triggering

Change the message in `app.py`, save, then run:

```powershell
git add .
git commit -m "Update application message"
git push
```

Wait for the next SCM polling interval and check Jenkins build history.

## Notes and troubleshooting

- This sample deploys locally to Docker, not AWS or GCP. Cloud deployment needs cloud resources, credentials, and a separate deployment stage.
- Jenkins needs access to the Docker daemon. If Docker commands fail inside Jenkins, inspect the socket mount and Docker Desktop Linux engine configuration.
- If port 5000 is occupied, stop the conflicting container or change the host-side port mapping.
- If a Jenkins container already exists, remove it before recreating it: `docker rm -f jenkins`.
- Do not expose Jenkins publicly without authentication, HTTPS, and appropriate security configuration.
