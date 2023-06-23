pipeline {
    agent any
    environment {
        DOCKERHUB_USERNAME = "prvinsm21"
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
        APP_NAME = "cicd-proj2"
        IMAGE_TAG = "${BUILD_NUMBER}"
        IMAGE_NAME = "${DOCKERHUB_USERNAME}" + "/" + "${APP_NAME}"
        REGISTRY_CREDS = 'dockerhub'
    }

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
                   docker_image = docker.build "${IMAGE_NAME}" 
                }
            }
        }
        stage('Login') {

			steps {
				sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
			}
		}
        stage ('Push image to DockerHub') { 
            steps {
                script {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }
       
        stage ('Delete Docker images') {
            steps {
                script {
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
                }
            }
        }
        stage ('Deployment stage') {
            steps {
                script {
                    def apply = false
                    try {
                        input message: 'Please confirm the apply to initiate the Deployment',ok: 'Ready to apply the config.'
                        apply = true
                    }
                    catch (err) {
                        apply = false
                        CurrentBuild.result = 'UNSTABLE'
                    }
                    if (apply) {
                        sh """
                            kubectl --as=macko apply -f .
                        """;
                    }
                }
            }
        }
    }
}
