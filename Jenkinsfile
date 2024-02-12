pipeline {
    agent any
    
    tools{
        jdk 'jdk17'
        maven 'maven3'
    }
    
    environment{
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', changelog: false, credentialsId: '512580bf-72d7-49e5-8d3b-e02d8cc2928a', poll: false, url: 'https://github.com/jideoni/Shopping-Cart.git'
            }
        }
        
        stage('COMPILE') {
            steps {
                sh "mvn clean compile -DskipTests=True"
            }
        }
        
        stage('OWASPScan') {
            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'DP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('SonarQube') {
            steps {
                withSonarQubeEnv('sonar-server'){
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Shopping-Cart \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=Shopping-Cart '''
                }
            }
        }
        
        stage('Build') {
            steps {
                sh "mvn clean compile -DskipTests=True"
            }
        }
        
        stage('Docker Build & Push') {
            steps {
                script{
                    withDockerRegistry(credentialsId: '954801ef-b723-43b8-9d02-f5404ba23a66', toolName: 'docker') {
                        
                        sh "docker build -t shopping-cart -f docker/Dockerfile ."
                        sh "docker tag shopping-cart jideoni/shopping-cart:latest"
                        sh "docker push jideoni/shopping-cart:latest"
                    }
                }
            }
        }
        
        stage('Deploy') {
            steps {
                script{
                    withDockerRegistry(credentialsId: '954801ef-b723-43b8-9d02-f5404ba23a66', toolName: 'docker') {
                        
                        sh "docker run -d --name shop-shop -p 8070:8070 jideoni/shopping-cart:latest"
                    }
                }
            }
        }
        
    }
}
