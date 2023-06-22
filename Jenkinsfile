pipeline {
    agent any

    stages {
        stage ('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/prvinsm21/demo-counter-app.git'
            }
        }
        stage ('Unit Testing') {
            steps {
                sh 'mvn test'
            }
        }
        stage ('Integration Testing') {
            steps {
                sh 'mvn verify -DskipUnitTests'
            }
        }
        stage ('Build stage') {
            steps {
                sh 'mvn clean install'
            }
        }
        stage ('Static Code analysis') {
            steps {
                script {
                withSonarQubeEnv(credentialsId: 'sonar-api') {
                sh 'mvn clean package sonar:sonar'
            }
            }
            }
        }
        stage ('Quality Gate status') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-api'
                }
            }
        }
        stage ('Upload jar files to nexus') {
            steps {
                script {
                    nexusArtifactUploader artifacts: [
                        [
                            artifactId: 'springboot', classifier: '', file: 'target/Uber.jar', type: 'jar'
                        ]
                        ], 
                        credentialsId: 'nexus-auth', 
                        groupId: 'com.example', 
                        nexusUrl: '192.168.29.38:8081/', 
                        nexusVersion: 'nexus3', 
                        protocol: 'http', 
                        repository: 'CICD-Proj2-release', 
                        version: '1.0.0'
                }
            }
        }
    }
}