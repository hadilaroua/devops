pipeline {
    agent any

    tools {
        maven 'M2_HOME'
    }

    environment {
        // Nexus Docker Hosted repo
        NEXUS_REGISTRY = "192.168.33.10:8085"
        IMAGE_NAME     = "devops-app"
        IMAGE_TAG      = "${BUILD_NUMBER}"

        // Jenkins credentials IDs
        NEXUS_CREDENTIALS = "nexus"
        SONAR_TOKEN_ID    = "sonarqube-token"
    }

    stages {

        stage('Checkout Git') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/hadilaroua/devops.git'
            }
        }

        stage('Build & Unit Tests') {
            steps {
                sh 'mvn clean test'
            }
        }

        stage('JUnit Reports') {
            steps {
                junit '**/target/surefire-reports/*.xml'
            }
        }

        stage('SonarQube Analysis') {
            environment {
                SONAR_TOKEN = credentials('sonarqube-token')
            }
            steps {
                sh '''
                  mvn sonar:sonar \
                  -Dsonar.projectKey=devops-pipeline \
                  -Dsonar.host.url=http://192.168.33.10:9000 \
                  -Dsonar.login=$SONAR_TOKEN
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                  docker build \
                  -t $NEXUS_REGISTRY/$IMAGE_NAME:$IMAGE_TAG \
                  -f docker/Dockerfile .
                '''
            }
        }

        stage('Push Image to Nexus') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: NEXUS_CREDENTIALS,
                    usernameVariable: 'NEXUS_USER',
                    passwordVariable: 'NEXUS_PASS'
                )]) {
                    sh '''
                      echo "$NEXUS_PASS" | docker login $NEXUS_REGISTRY \
                      -u "$NEXUS_USER" --password-stdin

                      docker push $NEXUS_REGISTRY/$IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline completed successfully'
        }
        failure {
            echo '❌ Pipeline failed – check logs'
        }
        always {
            cleanWs()
        }
    }
}
