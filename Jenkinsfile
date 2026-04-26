pipeline {
    agent any

    tools {
        nodejs "node18"
    }

    environment {
        DOCKER_CREDS = 'docker-hub'
        DOCKER_IMAGE = 'hashimaliii/sample-app' 
        // Note: SONAR_TOKEN is handled by the withSonarQubeEnv block
    }

    stages {
        // WE REMOVED THE CHECKOUT STAGE - Jenkins does it automatically!

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'sonar-scanner'
                    withSonarQubeEnv('SonarQube') {
                        sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=sample-app -Dsonar.sources=."
                    }
                }
            }
        }

        stage('Trivy Security Scan') {
            steps {
                sh "trivy fs . > trivy-scan-results.txt"
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('', DOCKER_CREDS) {
                        def myImage = docker.build("${DOCKER_IMAGE}:${env.BUILD_ID}")
                        myImage.push()
                        myImage.push('latest')
                    }
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'trivy-scan-results.txt', allowEmptyArchive: true
        }
    }
}
