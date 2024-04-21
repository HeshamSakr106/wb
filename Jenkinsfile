pipeline {
    agent any
    triggers {
        githubPush()
    }
    environment {
        DOCKERHUB_CREDENTIALS = credentials('Dockerhub')
    }
    stages {
        stage('Verify Branch') {
            steps {
                echo "$GIT_BRANCH"
            }
        }
        stage('Login to Dockerhub') {
            steps {
                script {
                    // Retrieve Dockerhub credentials from Jenkins
                    withCredentials([usernamePassword(credentialsId: 'Dockerhub', passwordVariable: 'DOCKERHUB_CREDENTIALS_PSW', usernameVariable: 'DOCKERHUB_CREDENTIALS_USR')]) {
                        // Log in to Dockerhub
                        sh'docker login -u ${DOCKERHUB_CREDENTIALS_USR} -p ${DOCKERHUB_CREDENTIALS_PSW}'
                    }
                }
            }
        }
        stage('Pulling base image from Dockerhub') {
            steps {
                sh'docker pull free5gc/webconsole-base:latest'
            }
        }
        stage('docker build') {
            steps {
                bat(script: """
                    docker images -a
                    docker build -t heshamsakr106/webui:latest . 
                    docker images -a
                """)
            }
        }
        stage('Scan Image for Common Vulnerabilities and Exposures') {
            steps {
                sh'docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image heshamsakr106/webui:latest'
            }
        }

        stage('Pushing to Dockerhub') {
            steps {
                sh'docker push heshamsakr106/webui:latest'
            }
        }
        stage('Build and Package Helm Chart') {
            steps {
                sh'helm package ./helm/'
            }
        }
        stage('Configure Kubernetes Context') {
            steps {
                sh'aws eks --region us-east-1 update-kubeconfig --name 5G-Core-Net'
            }
        }
        stage('Deploy Helm Chart on EKS') {
            steps {
                sh'helm upgrade --install amf ./helm/'
            }
        }
    }
    post {
        always {
            // Archiving Test Result
            archiveArtifacts artifacts: 'trivy-report.json', fingerprint: true
            sh'docker rmi heshamsakr106/webui:latest'
            sh'docker rmi free5gc/webconsole-base:latest'
        }
    }
}