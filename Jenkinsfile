pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = "s224021028acr.azurecr.io"
        DOCKER_IMAGE_1 = "stories-backend"
        DOCKER_IMAGE_2 = "stories-frontend"
        KUBECONFIG = "C:/Users/White/.kube/config"
    }

    stages {

        stage("Build") {
            steps {
                script {
                    dir("./server") {
                        docker.build("${DOCKER_IMAGE_1}")
                    }
                    dir("./client") {
                        docker.build("${DOCKER_IMAGE_2}")
                    }
                }
            }
        }

        stage("Test") {
            steps {
                script {
                    dir("./client") {
                        bat "npm install"
                        bat "npm test"
                    }
                    dir("./server") {
                        bat "npm install"
                        bat "npm test"
                    }
                }
            }
        }

        stage("Code Quality Analysis") {
            steps {
                script {
                    def scannerHome = tool "SonarScanner";
                    withSonarQubeEnv("SonarQube") {
                        bat "${scannerHome}/bin/sonar-scanner"
                    }
                }
            }
        }

        stage("Push Docker Image") {
            steps {
                script {
                    docker.withRegistry("https://${DOCKER_REGISTRY}", "c8aa2f93-4fdf-4bb9-a36d-fa57c063edb0") {
                        bat "docker tag ${DOCKER_IMAGE_1} ${DOCKER_REGISTRY}/${DOCKER_IMAGE_1}"
                        bat "docker tag ${DOCKER_IMAGE_2} ${DOCKER_REGISTRY}/${DOCKER_IMAGE_2}"
                        bat "docker push ${DOCKER_REGISTRY}/${DOCKER_IMAGE_1}"
                        bat "docker push ${DOCKER_REGISTRY}/${DOCKER_IMAGE_2}"
                    }
                }
            }
        }

        stage("Deploy to Staging Environment with Docker Compose") {
            steps {
                script {
                    bat "docker compose up -d"
                }
            }
        }

        stage("Release to Production on AKS") {
            steps {
                script {
                    bat "helm repo add bitnami https://charts.bitnami.com/bitnami --kubeconfig ${KUBE_CONFIG}"
                    bat "helm upgrade --install stories-mongo --set auth.enabled=false,arbiter.enabled=false,architecture=replicaset,replicaSet.replicas.secondary=1 bitnami/mongodb --kubeconfig ${KUBE_CONFIG}"
                    bat "helm upgrade --install stories-backend ./helm-charts/backend --kubeconfig ${KUBE_CONFIG}"
                    bat "helm upgrade --install stories-frontend ./helm-charts/frontend --kubeconfig ${KUBE_CONFIG}"
                }
            }
        }

        stage("Monitoring and Alerting using Grafana") {
            steps {
                script {
                    bat "helm repo add bitnami https://charts.bitnami.com/bitnami --kubeconfig ${KUBE_CONFIG}"
                    bat "helm upgrade --install stories-grafana bitnami/grafana --set service.type=LoadBalancer --kubeconfig ${KUBE_CONFIG}"
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}