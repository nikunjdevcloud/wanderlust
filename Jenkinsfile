pipeline {
    agent any
    environment {
        SONAR_HM = tool "Sonar"
    }
    stages {
        stage('Code Clone from GitHub: Step-1') {
            steps {
                echo 'Cloning code from GitHub'
                git url: 'https://github.com/nikunjdevcloud/wanderlust.git', branch: 'devops'
            }
        }
        stage('SonarQube Analysis: Step-2') {
            steps {
                withSonarQubeEnv("Sonar") {
                    sh "$SONAR_HM/bin/sonar-scanner -Dsonar.projectName=Wanderlust -Dsonar.projectKey=Wanderlust -X"
                    echo 'SonarQube Analysis Done'
                }
            }
        }
        stage('Sonar Quality Gates: Step-3') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: false
                }
                echo 'Quality Gates of Code Completed.'
            }
        }
        stage('OWASP: Step-4') {
            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'OWASP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                echo 'OWASP Dependency check completed.'
            }
        }
        stage('Code Build and Test: Step-5') {
            steps {
                echo 'Building Docker image'
                sh 'docker build -t wanderlust-back-node:v1 ./backend'
                echo 'Backend Docker image build done.'
                sh 'docker build -t wanderlust-front-react:v2 ./frontend'
                echo 'Frontend Docker image build done.'
            }
        }
        stage('Trivi Image Scan: Step-6') {
            steps {
                sh 'trivy image wanderlust-back-node:v1'
                sh 'trivy image wanderlust-front-react:v2'
                echo 'Images Scan Completed.'
            }
        }
        stage('Image Push to Docker Hub: Step-7') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'DockerHubCredentials',
                    passwordVariable: 'dockerHubPass',
                    usernameVariable: 'dockerHubUser'
                )]) {
                    echo 'Logging in to Docker Hub'
                    sh 'docker login -u ${dockerHubUser} -p ${dockerHubPass}'
                    echo 'Tagging Backend Docker image'
                    sh 'docker tag wanderlust-back-node:v1 ${dockerHubUser}/wanderdevsec:Backend'
                    echo 'Pushing Backend Docker image to Docker Hub'
                    sh 'docker push ${dockerHubUser}/wanderdevsec:Backend'

                    echo 'Tagging Frontend Docker image'
                    sh 'docker tag wanderlust-front-react:v2 ${dockerHubUser}/wanderdevsec:Frontend'
                    echo 'Pushing Frontend Docker image to Docker Hub'
                    sh 'docker push ${dockerHubUser}/wanderdevsec:Frontend'
                    echo 'Both Frontend and Backend Docker images Pushed to DockerHub Successfully Completed..!!'
                }
            }
        }
        stage('Deploy with Docker Compose: Step-8') {
            steps {
                echo 'Deploying with Docker Compose'
                sh 'docker-compose down'
                sh 'docker-compose up -d'
            }
        }
    }
}
