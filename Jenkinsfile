pipeline {
    agent any

    tools {
        jdk 'jdk'
        maven 'maven3'
    }

    environment {
        DOCKER_IMAGE = 'darshanpatil0507/demo'
        SONAR_PROJECT_KEY = 'new1'
        SONAR_PROJECT_NAME = 'new'
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Compile') {
            steps {
                sh 'mvn -B clean compile'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        ${SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=$SONAR_PROJECT_KEY \
                        -Dsonar.projectName=$SONAR_PROJECT_NAME \
                        -Dsonar.java.binaries=target/classes
                    '''
                }
            }
        }

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
                    )
                ]) {
                    sh '''
                        set -e
                        echo "$DOCKERHUB_PASSWORD" | docker login \
                        -u "$DOCKERHUB_USERNAME" --password-stdin

                        docker push "$DOCKER_IMAGE:$BUILD_NUMBER"
                        docker tag "$DOCKER_IMAGE:$BUILD_NUMBER" "$DOCKER_IMAGE:latest"
                        docker push "$DOCKER_IMAGE:latest"
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([
                    file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG_FILE')
                ]) {
                    sh '''
                        set -e
                        export KUBECONFIG="$KUBECONFIG_FILE"

                        sed -i "s|IMAGE_PLACEHOLDER|$DOCKER_IMAGE:$BUILD_NUMBER|g" k8s/deployment.yaml

                        kubectl apply -f k8s/deployment.yaml

                        kubectl rollout status deployment/demoapp --timeout=180s

                        kubectl get pods -o wide
                        kubectl get services
                    '''
                }
            }
        }
    }

    post {
        always {
            sh 'docker image prune -f || true'
            echo 'Pipeline execution completed.'
        }
        success {
            echo 'SUCCESS: CI/CD pipeline completed successfully.'
        }
        failure {
            echo 'FAILURE: Check the stage console output.'
        }
    }
}