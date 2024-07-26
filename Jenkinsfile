pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS_ID = 'docker-hub-credentials-id'
        DOCKER_HUB_REPO = credentials('docker-hub-juice-shop-repo-id')
        DOCKER_BUILDKIT = '1'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Get Commit ID') {
            steps {
                script {
                    COMMIT_ID = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    echo "Building Docker image for commit ID: ${COMMIT_ID}"
                }
            }
        }
        stage('Docker Build') {
            steps {
                script {
                    docker.build("${DOCKER_HUB_REPO}:latest", ".")
                }
            }
        }
        stage('Docker Push') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_HUB_CREDENTIALS_ID}") {
                        docker.image("${DOCKER_HUB_REPO}:latest").push()
                    }
                }
            }
        }
        stage('Analyze Image') {
            steps {

                withCredentials([usernamePassword(credentialsId: "${DOCKER_HUB_CREDENTIALS_ID}", usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    script {
                        // Install Docker Scout
                        sh 'curl -sSfL https://raw.githubusercontent.com/docker/scout-cli/main/install.sh | sh -s -- -b /usr/local/bin'

                        // Log into Docker Hub
                        sh """
                        echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin'
                        """

                        // Analyze and fail on critical or high vulnerabilities
                        sh """
                        docker-scout cves "${DOCKER_HUB_REPO}:latest" --exit-code --only-severity critical,high
                        """
                    }
                }
            }
        }
    }
}
