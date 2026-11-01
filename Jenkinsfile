pipeline {
    agent any

    tools {
        jdk 'jdk'
        maven 'maven3'
    }

    environment {
        DOCKER_IMAGE = 'darshanpatil0507/demo'
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {

        stage('Git Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Code Compile') {
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        ${SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=new1 \
                        -Dsonar.projectName=new \
                        -Dsonar.java.binaries=target/classes
                    '''
                }
            }
        }

        stage('Code Build') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build \
                    -t $DOCKER_IMAGE:$BUILD_NUMBER .
                '''
            }
        }

        stage('Docker Image Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'Dockerhub_cred',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        set -e

                        echo "$DOCKER_PASS" | docker login \
                        --username "$DOCKER_USER" \
                        --password-stdin

                        docker push $DOCKER_IMAGE:$BUILD_NUMBER

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([
                    file(
                        credentialsId: 'kubeconfig',
                        variable: 'KUBECONFIG'
                    )
                ]) {
                    sh '''
                        set -e

                        echo "Deploying application to Kubernetes..."

                        sed -i "s|IMAGE_PLACEHOLDER|$DOCKER_IMAGE:$BUILD_NUMBER|g" k8s/deployment.yaml

                        kubectl apply -f k8s/deployment.yaml

                        echo "Checking deployed Docker image..."

                        kubectl get deployment demoapp \
                        -o=jsonpath='{.spec.template.spec.containers[*].image}'

                        echo

                        echo "Waiting for Kubernetes rollout..."

                        kubectl rollout status deployment/demoapp \
                        --timeout=180s

                        echo "Kubernetes Pods:"
                        kubectl get pods -o wide

                        echo "Kubernetes Services:"
                        kubectl get services
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: CI/CD Pipeline completed successfully!'
        }

        failure {
            echo 'FAILURE: Check Jenkins Console Output.'
        }

        always {
            echo 'Jenkins Pipeline execution completed.'
        }
    }
}
