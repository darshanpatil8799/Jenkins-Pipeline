pipeline {
    agent any
    tools {
        jdk 'jdk'
        maven 'maven3'
    }
    environment {
        // Replace with your Docker Hub username/repository.
        DOCKER_IMAGE = 'YOUR_DOCKERHUB_USERNAME/demoapp'
        SONAR_PROJECT_KEY = 'demoapp'
        SONAR_PROJECT_NAME = 'demoapp'
    }
    stages {
        stage('Git Checkout') {
            steps { checkout scm }
        }
        stage('Compile') {
            steps { sh 'mvn -B clean compile' }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                      mvn -B sonar:sonar \\
                        -Dsonar.projectKey=$SONAR_PROJECT_KEY \\
                        -Dsonar.projectName=$SONAR_PROJECT_NAME
                    '''
                }
            }
        }
        stage('Build and Test') {
            steps { sh 'mvn -B clean package' }
            post {
                always { junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml' }
            }
        }
        stage('Docker Build') {
            steps { sh 'docker build -t $DOCKER_IMAGE:$BUILD_NUMBER .' }
        }
        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKERHUB_USERNAME',
                    passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                    sh '''
                      echo "$DOCKERHUB_PASSWORD" | docker login -u "$DOCKERHUB_USERNAME" --password-stdin
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
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG_FILE')]) {
                    sh '''
                      export KUBECONFIG="$KUBECONFIG_FILE"
                      sed "s|YOUR_DOCKERHUB_USERNAME/demoapp:latest|$DOCKER_IMAGE:$BUILD_NUMBER|g" k8s/deployment.yaml | kubectl apply -f -
                      kubectl rollout status deployment/demoapp --timeout=180s
                    '''
                }
            }
        }
    }
    post {
        always { sh 'docker image prune -f || true' }
        success { echo 'CI/CD pipeline completed successfully.' }
        failure { echo 'Pipeline failed. Check the stage console output.' }
    }
}
