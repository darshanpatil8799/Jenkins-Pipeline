pipeline {
    agent any

    tools {
        jdk 'jdk'
        maven 'maven3'
    }

    environment {
        DOCKER_IMAGE = 'darshanpatil0507/demo'
<<<<<<< HEAD
        SONAR_PROJECT_KEY = 'new1'
        SONAR_PROJECT_NAME = 'new'
=======
>>>>>>> 9d4acd4fc0013aade1984053da99f3bde5732a10
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {

        stage('Git Checkout') {
            steps {
                checkout scm
            }
        }

<<<<<<< HEAD
        stage('Compile') {
            steps {
                sh 'mvn -B clean compile'
=======
        stage('Code Compile') {
            steps {
                sh 'mvn clean compile'
>>>>>>> 9d4acd4fc0013aade1984053da99f3bde5732a10
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        ${SCANNER_HOME}/bin/sonar-scanner \
<<<<<<< HEAD
                        -Dsonar.projectKey=$SONAR_PROJECT_KEY \
                        -Dsonar.projectName=$SONAR_PROJECT_NAME \
=======
                        -Dsonar.projectKey=new1 \
                        -Dsonar.projectName=new \
>>>>>>> 9d4acd4fc0013aade1984053da99f3bde5732a10
                        -Dsonar.java.binaries=target/classes
                    '''
                }
            }
        }

<<<<<<< HEAD
        stage('Build and Test') {
            steps {
                sh 'mvn -B clean package'
            }
            post {
                always {
                    junit allowEmptyResults: true,
                          testResults: 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE:$BUILD_NUMBER .'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'Dockerhub_cred',
                        usernameVariable: 'DOCKERHUB_USERNAME',
                        passwordVariable: 'DOCKERHUB_PASSWORD'
=======
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
>>>>>>> 9d4acd4fc0013aade1984053da99f3bde5732a10
                    )
                ]) {
                    sh '''
                        set -e
<<<<<<< HEAD
                        echo "$DOCKERHUB_PASSWORD" | docker login \
                        -u "$DOCKERHUB_USERNAME" --password-stdin

                        docker push "$DOCKER_IMAGE:$BUILD_NUMBER"
                        docker tag "$DOCKER_IMAGE:$BUILD_NUMBER" "$DOCKER_IMAGE:latest"
                        docker push "$DOCKER_IMAGE:latest"
=======

                        echo "$DOCKER_PASS" | docker login \
                        --username "$DOCKER_USER" \
                        --password-stdin

                        docker push $DOCKER_IMAGE:$BUILD_NUMBER

>>>>>>> 9d4acd4fc0013aade1984053da99f3bde5732a10
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([
<<<<<<< HEAD
                    file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG_FILE')
                ]) {
                    sh '''
                        set -e
                        export KUBECONFIG="$KUBECONFIG_FILE"
=======
                    file(
                        credentialsId: 'kubeconfig',
                        variable: 'KUBECONFIG'
                    )
                ]) {
                    sh '''
                        set -e

                        echo "Deploying application to Kubernetes..."
>>>>>>> 9d4acd4fc0013aade1984053da99f3bde5732a10

                        sed -i "s|IMAGE_PLACEHOLDER|$DOCKER_IMAGE:$BUILD_NUMBER|g" k8s/deployment.yaml

                        kubectl apply -f k8s/deployment.yaml

<<<<<<< HEAD
                        kubectl rollout status deployment/demoapp --timeout=180s

                        kubectl get pods -o wide
=======
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
>>>>>>> 9d4acd4fc0013aade1984053da99f3bde5732a10
                        kubectl get services
                    '''
                }
            }
        }
    }

    post {
<<<<<<< HEAD
        always {
            sh 'docker image prune -f || true'
            echo 'Pipeline execution completed.'
        }
        success {
            echo 'SUCCESS: CI/CD pipeline completed successfully.'
        }
        failure {
            echo 'FAILURE: Check the stage console output.'
=======
        success {
            echo 'SUCCESS: CI/CD Pipeline completed successfully!'
        }

        failure {
            echo 'FAILURE: Check Jenkins Console Output.'
        }

        always {
            echo 'Jenkins Pipeline execution completed.'
>>>>>>> 9d4acd4fc0013aade1984053da99f3bde5732a10
        }
    }
}