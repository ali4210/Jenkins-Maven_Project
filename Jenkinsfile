pipeline {
    agent any

    parameters {
        choice(name: 'ACTION', choices: ['Deploy', 'Destroy'], description: 'Choose Action')
    }

    tools {
        maven 'MyLocalMaven'
        jdk 'JDK-17'
        // Scanner removed from here to prevent syntax errors
    }

    environment {
        IMAGE_NAME = "my-java-app"
        APP_PORT = "8090"
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build JAR') {
            when { expression { params.ACTION == 'Deploy' } }
            steps {
                sh 'mvn clean package'
            }
        }

        stage('SonarQube Analysis') {
            when { expression { params.ACTION == 'Deploy' } }
            steps {
                script {
                    // BRUTE FORCE AUTHENTICATION
                    
                    // 1. Point to the manual installation folder
                    def scannerHome = '/opt/sonar-scanner'
                    
                    // 2. Run the command with the TOKEN directly in the string
                    // YOU MUST REPLACE 'PASTE_YOUR_TOKEN_HERE' WITH YOUR ACTUAL TOKEN
                    sh """
                        ${scannerHome}/bin/sonar-scanner \
                        -Dsonar.login=squ_6d5251468ad325f36953a8d6f83285f3d4b1bddf \
                        -Dsonar.projectKey=my-java-app \
                        -Dsonar.sources=src \
                        -Dsonar.java.binaries=target/classes
                    """
                }
            }
        }

        stage('Build Docker Image') {
            when { expression { params.ACTION == 'Deploy' } }
            steps {
                script {
                   def dockerfileContent = """
                       FROM eclipse-temurin:17-jdk-alpine
                       COPY target/*.jar app.jar
                       ENTRYPOINT ["java", "-jar", "/app.jar"]
                   """
                   writeFile file: 'Dockerfile', text: dockerfileContent
                }
                sh "docker build -t ${IMAGE_NAME} ."
            }
        }

        stage('Deploy Container') {
            when { expression { params.ACTION == 'Deploy' } }
            steps {
                sh "docker stop ${IMAGE_NAME} || true"
                sh "docker rm ${IMAGE_NAME} || true"
                sh "docker run -d -p ${APP_PORT}:8080 --name ${IMAGE_NAME} ${IMAGE_NAME}"
            }
        }

        stage('Destroy Container') {
            when { expression { params.ACTION == 'Destroy' } }
            steps {
                sh "docker stop ${IMAGE_NAME} || true"
                sh "docker rm ${IMAGE_NAME} || true"
            }
        }
    }
}
