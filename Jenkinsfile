pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = "s224021028acr.azurecr.io"
        DOCKER_IMAGE_1 = "stories-backend"
        DOCKER_IMAGE_2 = "stories-frontend"
        //KUBECONFIG = '/path/to/your/kubeconfig'
    }

    stages {

        stage("Build") {
            steps {
                script {
                    dir("./server") {
                        docker.build("${DOCKER_IMAGE_1}:latest")
                    }
                    dir("./client") {
                        docker.build("${DOCKER_IMAGE_2}:latest")
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

        /*stage("Code Quality Analysis") {
            steps {
                script {
                    def scannerHome = tool "SonarScanner";
                    withSonarQubeEnv("SonarQube") {
                        bat "${scannerHome}/bin/sonar-scanner"
                    }
                }
            }
        }*/

        stage("Push Docker Image") {
            steps {
                script {
                    docker.withRegistry("https://${DOCKER_REGISTRY}", "c8aa2f93-4fdf-4bb9-a36d-fa57c063edb0") {
                        def image1 = docker.image("${DOCKER_IMAGE_1}:latest")
                        def image2 = docker.image("${DOCKER_IMAGE_2}:latest")
                        image1.tag("${DOCKER_REGISTRY}/${DOCKER_IMAGE_1}:latest")
                        image2.tag("${DOCKER_REGISTRY}/${DOCKER_IMAGE_2}:latest")
                        image1.push()
                        image2.push()
                    }
                }
            }
        }

        stage("Deploy to Staging Environment with Docker Compose") {
            steps {
                script {
                    bat "docker compose up"
                }
            }
        }

        /*stage('Release to Production') {
            steps {
                script {
                    bat "helm repo add bitnami https://charts.bitnami.com/bitnami"
                    bat "helm upgrade --install stories-mongo --set auth.enabled=false,arbiter.enabled=false,architecture=replicaset,replicaSet.replicas.secondary=1 bitnami/mongodb --kubeconfig="
                    dir("./helm-charts/backend") {
                        bat "helm upgrade --install stories-backend --kubeconfig="
                    }
                    dir("./helm-charts/frontend") {
                        bat "helm upgrade --install stories-frontend --kubeconfig="
                    }
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