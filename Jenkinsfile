pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = "s224021028acr.azurecr.io"
        DOCKER_IMAGE_1 = "stories-backend"
        DOCKER_IMAGE_2 = "stories-frontend"
        //KUBECONFIG = '/path/to/your/kubeconfig'
        //HELM_RELEASE_NAME = 'mern-app-release'
        //HELM_CHART_PATH = 'mern-chart/'
    }

    stages {

        stage('Build') {
            steps {
                script {
                    sh "cd server"
                    docker.build("${DOCKER_REGISTRY}/${DOCKER_IMAGE_1}:latest")
                    sh "cd .."
                    sh "cd client"
                    docker.build("${DOCKER_REGISTRY}/${DOCKER_IMAGE_2}:latest")
                }
            }
        }

        /*stage('Test') {
            steps {
                script {
                    sh 'npm install'
                    sh 'npm test'
                }
            }
        }*/

        stage("Code Quality Analysis") {
            steps {
                script {
                    def scannerHome = tool "SonarScanner";
                    withSonarQubeEnv() {
                        sh "${scannerHome}/bin/sonar-scanner"
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry("https://${DOCKER_REGISTRY}", "384f8bd0-a2a3-4868-a88e-b05fd19f3fac") {
                        docker.image("${DOCKER_REGISTRY}/${DOCKER_IMAGE_1}:latest").push()
                        docker.image("${DOCKER_REGISTRY}/${DOCKER_IMAGE_2}:latest").push()
                    }
                }
            }
        }

        /*stage('Deploy to AKS with Helm') {
            steps {
                script {
                    sh """
                    helm upgrade --install ${HELM_RELEASE_NAME} ${HELM_CHART_PATH} \
                    --set image.repository=${DOCKER_REGISTRY}/${DOCKER_IMAGE} \
                    --set image.tag=latest \
                    --kubeconfig=${KUBECONFIG}
                    """
                }
            }
        }

        stage('Release to Production') {
            steps {
                script {
                    // Helm promotes application to production
                    sh """
                    helm upgrade --install ${HELM_RELEASE_NAME}-prod ${HELM_CHART_PATH} \
                    --set image.repository=${DOCKER_REGISTRY}/${DOCKER_IMAGE} \
                    --set image.tag=latest \
                    --kubeconfig=${KUBECONFIG} \
                    --namespace production
                    """
                }
            }
        }

        stage('Monitoring & Alerting') {
            steps {
                script {
                    // Monitor using Datadog or New Relic
                    sh 'datadog-agent monitor your-service'
                }
            }
        }*/
    }

    post {
        always {
            cleanWs()
        }
    }
}