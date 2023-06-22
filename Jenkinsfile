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
                    def readPomVersion = readMavenPom file: 'pom.xml'
                    def nexusRepo = readPomVersion.version.endsWith("SNAPSHOT") ? "cicd-proj2-snapshot" : "cicd-proj2-release"
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
                        repository: nexusRepo, 
                        version: "${readPomVersion.version}"
                }
            }
        }
        stage (' Docker image build') {
            steps {
                script {
                    sh 'docker image build -t cicd-proj2:v1.$BUILD_ID .'
                    sh 'docker image tag cicd-proj2:v1.$BUILD_ID prvinsm21/cicd-proj2:v1.$BUILD_ID'
                    sh 'docker image tag cicd-proj2:v1.$BUILD_ID prvinsm21/cicd-proj2:latest'
                }
            }
        }
        stage ('Push image to DockerHub') {
            steps {
                script {
                    withCredentials([usernameColonPassword(credentialsId: '507c49df-3053-4069-a545-23ccdda3935d', variable: 'dockerhub-cred')])  {
                        sh 'docker login -u prvinsm21 -p ${dockerhub-cred}'
                        sh 'docker image push prvinsm21/cicd-proj2:v1.$BUILD_ID'
                        sh 'docker image push prvinsm21/cicd-proj2:latest'
                    }
                }
            }
        }
    }
}