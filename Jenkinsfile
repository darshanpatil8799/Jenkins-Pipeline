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
                        -Dsonar.projectName=new
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
                sh 'docker build -t $DOCKER_IMAGE:$BUILD_ID .'
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
                        echo "$DOCKER_PASS" | docker login \
                        -u "$DOCKER_USER" --password-stdin

                        docker push $DOCKER_IMAGE:$BUILD_ID

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([
                    file(credentialsId: 'kubeconfig',
                         variable: 'KUBECONFIG')
                ]) {
                    sh '''
                        sed -i "s|IMAGE_PLACEHOLDER|$DOCKER_IMAGE:$BUILD_ID|g" k8s/deployment.yaml

                        kubectl apply -f k8s/deployment.yaml

                        kubectl rollout status deployment/demoapp --timeout=180s

                        kubectl get pods -o wide
                        kubectl get services
                    '''
                }
            }
        }
    }
}
