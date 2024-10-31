pipeline {
    agent any
    environment {
        SONARQUBE_SERVER = 'http://192.168.1.18:9000' // Update with actual SonarQube IP
        SONARQUBE_AUTH_TOKEN = credentials('sqa_f5125389d820feb78c7858ce294a5987d348856c') // SonarQube token stored in Jenkins credentials
        SNYK_TOKEN = credentials('b8a35b22-ecd3-4f69-bb9f-0636f6aac5f8') // Snyk API token stored in Jenkins credentials
    }
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/CharanRayudu/juice-shop.git'
            }
        }
        
        stage('Build') {
            steps {
                script {
                    // Builds the Docker image for Juice Shop
                    sh 'docker-compose build'
                }
            }
        }
        
        stage('SAST - SonarQube Analysis') {
            steps {
                script {
                    // Run SonarQube scan
                    sh """
                    sonar-scanner \
                      -Dsonar.projectKey=JuiceShop \
                      -Dsonar.sources=. \
                      -Dsonar.host.url=${SONARQUBE_SERVER} \
                      -Dsonar.login=${SONARQUBE_AUTH_TOKEN}
                    """
                }
            }
        }
        
        stage('SCA - Snyk Vulnerability Scan') {
            steps {
                script {
                    // Run Snyk to analyze dependencies
                    sh 'docker run --rm -v $(pwd):/project -e SNYK_TOKEN=${SNYK_TOKEN} snyk/snyk-cli test --all-projects'
                }
            }
        }
        
        stage('DAST - OWASP ZAP') {
            steps {
                script {
                    // Run OWASP ZAP to scan the deployed app
                    sh """
                    docker run -v $(pwd):/zap/wrk/:rw -t zaproxy/zap-stable zap-baseline.py \
                      -t http://localhost:3000 -r zap_report.html
                    """
                }
                archiveArtifacts artifacts: 'zap_report.html', allowEmptyArchive: true
            }
        }
        
        stage('Image Scan - Trivy') {
            steps {
                script {
                    // Run Trivy to scan the Docker image
                    sh 'docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image juicestore:latest'
                }
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    // Deploy the vulnerable app using Docker Compose
                    sh 'docker-compose up -d'
                }
            }
        }
    }
    
    post {
        always {
            script {
                // Clean up Docker containers and images to free space after the build
                sh 'docker-compose down'
                sh 'docker system prune -f'
            }
        }
    }
}
