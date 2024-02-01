def imageName="192.168.44.44:8082/docker_registry/backend"
def dockerTag=""
def dockerRegistry="https://192.168.44.44:8082"
def registryCredentials="artifactory"

pipeline {
    agent {
        label 'agent'
    }

    environment {
        PIP_BREAK_SYSTEM_PACKAGES = 1
        scannerHome = tool 'SonarQube'
    }

    stages {
        stage('Get Code') {
            steps {
                checkout scm
            }
        }

        stage('Unit tests') {
            steps {
                sh "pip3 install -r requirements.txt"
                sh "python3 -m pytest --cov=. --cov-report xml:test-results/coverage.xml --junitxml=test-results/pytest-report.xml"
            }
        }
        
        stage('Sonarqube analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }
        
        stage('Build application image') {
            steps {
                script {
                  dockerTag = "RC-${env.BUILD_ID}.${env.GIT_COMMIT.take(7)}"
                  applicationImage = docker.build("$imageName:$dockerTag",".")
                }
            }
        }

        stage ('Pushing image to docker registry') {
            steps {
                script {
                    docker.withRegistry("$dockerRegistry", "$registryCredentials") {
                        applicationImage.push()
                        applicationImage.push('latest')
                    }
                }
            }
        } 
    
    }

    post {
        always {
            junit testResults: "test-results/*.xml"
            cleanWs()
        }
    }
}
