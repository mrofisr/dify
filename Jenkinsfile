pipeline {
    agent any
    stages {
        stage('Info') {
            steps {
                echo 'Starting the pipeline...'
            }
        }
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Parse Config') {
            steps {
                script {
                    def configFile = 'build.config'
                    def yqCommand = { String query -> sh(script: "yq -C ${query} ${configFile}", returnStdout: true).trim() }
                    APP_NAME = yqCommand(".config.app.name")
                    APP_DESCRIPTION = yqCommand(".config.app.description")
                    MAINTAINER = yqCommand(".config.maintainer")
                    MODULES = yqCommand(".config.app.modules")
                    GIT_TAG = sh(script: "git describe --tags --match 'v[0-9]*' --abbrev=0 || echo 'latest'", returnStdout: true).trim()
                    echo "App Name: ${APP_NAME}"
                    echo "App Description: ${APP_DESCRIPTION}"
                    echo "Maintainer: ${MAINTAINER}"
                    echo "Modules: ${MODULES}"
                    echo "Version: ${GIT_TAG}"
                }
            }
        }
        stage('Build') {
            steps {
                script {
                    if (MODULES.contains('web') && MODULES.contains('api')) {
                        echo 'Building both Web and API Docker images...'
                        sh "docker build -t ghcr.io/mrofisr/dify-web:${GIT_TAG} -f web"
                        sh "docker build -t ghcr.io/mrofisr/dify-api:${GIT_TAG} -f api"
                    } else if (MODULES.contains('web')) {
                        echo 'Building Web Docker image...'
                        sh "docker build -t ghcr.io/mrofisr/dify-web:${GIT_TAG} -f web"
                    } else if (MODULES.contains('api')) {
                        echo 'Building API Docker image...'
                        sh "docker build -t ghcr.io/mrofisr/dify-api:${GIT_TAG} -f api"
                    } else {
                        error "No valid module found to build"
                    }
                }
            }
        }
        stage('Docker Login') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'container_registry', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASSWORD')]) {
                        echo "Logging in to Docker registry ${DOCKER_REGISTRY}"
                        sh "echo $DOCKER_PASSWORD | docker login ${DOCKER_REGISTRY} -u $DOCKER_USER --password-stdin"
                    }
                }
            }
        }
        stage('Image Push') {
            steps {
                script {
                    if (MODULES.contains('web') && MODULES.contains('api')) {
                        echo 'Pushing both Web and API Docker images...'
                        sh "docker push ghcr.io/mrofisr/dify-web:${GIT_TAG}"
                        sh "docker push ghcr.io/mrofisr/dify-api:${GIT_TAG}"
                    } else if (MODULES.contains('web')) {
                        echo 'Pushing Web Docker image...'
                        sh "docker push ghcr.io/mrofisr/dify-web:${GIT_TAG}"
                    } else if (MODULES.contains('api')) {
                        echo 'Pushing API Docker image...'
                        sh "docker push ghcr.io/mrofisr/dify-api:${GIT_TAG}"
                    } else {
                        error "No valid module found to push"
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh "git clone https://github.com/mrofisr/dify-kubernetes"
                    if (MODULES.contains('web') && MODULES.contains('api')) {
                        echo 'Deploying both Dify Web and Dify API App...'
                        sh "sed -i 's|{{VERSION}}|${GIT_TAG}|g' dify-kubernetes/dify-web/deployment.yaml"
                        sh "kubectl apply -f dify-kubernetes/dify-web/deployment.yaml"
                        sh "sed -i 's|{{VERSION}}|${GIT_TAG}|g' dify-kubernetes/dify-api/deployment.yaml"
                        sh "kubectl apply -f dify-kubernetes/dify-api/deployment.yaml"
                    } else if (MODULES.contains('web')) {
                        echo 'Deploying Dify Web App...'
                        sh "sed -i 's|{{VERSION}}|${GIT_TAG}|g' dify-kubernetes/dify-web/deployment.yaml"
                        sh "kubectl apply -f dify-kubernetes/dify-web/deployment.yaml"
                    } else if (MODULES.contains('api')) {
                        echo 'Deploying Dify API...'
                        sh "sed -i 's|{{VERSION}}|${GIT_TAG}|g' dify-kubernetes/dify-api/deployment.yaml"
                        sh "kubectl apply -f dify-kubernetes/dify-api/deployment.yaml"
                    } else {
                        error "No valid module found to push"
                    }
                }
            }
        }
    }
    post {
        success {
            echo "I will only say Hello if the pipeline is successful!"
        }
        failure {
            echo "I will only say Hello if the pipeline has failed!"
        }
    }
}
