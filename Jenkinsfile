pipeline {
    agent any

    triggers {
        pollSCM('H/2 * * * *')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo 'Source code checked out successfully'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'python3 -m pip install --break-system-packages -r requirements.txt'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'python3 -m pytest -v test_app.py'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t jenkins-cicd-demo:latest .'
            }
        }

        stage('Deploy Container Locally') {
            steps {
                sh '''
                    docker stop demo-app || true
                    docker rm demo-app || true
                    docker run -d --name demo-app -p 5000:5000 jenkins-cicd-demo:latest
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh 'docker ps --filter name=demo-app'
                sh 'docker logs --tail 20 demo-app'
            }
        }
    }

    post {
        success {
            echo 'CI/CD pipeline completed successfully!'
        }
        failure {
            echo 'CI/CD pipeline failed. Check the stage logs above.'
        }
    }
}
