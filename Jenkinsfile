//Initial Version Used for CI

/*pipeline {
    // 'agent any' means run this on any available server (node)
    agent any

    // => This section links the tools we configured earlier
    // CRITICAL: These names must match EXACTLY what you typed in Global Tool Configuration
    tools {
        maven 'MyLocalMaven' 
        jdk 'JDK-17' 
    }

    stages {
        // => Stage 1: Get the code from GitHub
        stage('Checkout Code') {
            steps {
                echo 'Downloading source code from GitHub...'
               	checkout scm
            }
        }

        // => Stage 2: Compile and Package
        stage('Build & Test') {
            steps {
                echo 'Compiling and Building JAR file...'
                // 'sh' runs a shell command on your Linux terminal
                sh 'mvn clean package'
            }
        }
    }

    // => This runs after the build finishes
    post {
        success {
            echo 'Build successful! Archiving the artifacts...'
            // This saves the .jar file so you can download it
            archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
        }
        failure {
            echo 'Build failed. Please check the logs.'
        }
    }
}
*/

//Modified Version with CD by using Docker to complete the process of CI/CD Pipeline

pipeline { agent any

tools {
    maven 'MyLocalMaven'
    jdk 'JDK-17'
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
        steps {
            sh 'mvn clean package'
        }
    }

    stage('Build Docker Image') {
        steps {
            script {
                def dockerfileContent = """
                    FROM openjdk:17-jdk-alpine
                    COPY target/*.jar app.jar
                    ENTRYPOINT ["java", "-jar", "/app.jar"]
                """
                writeFile file: 'Dockerfile', text: dockerfileContent
            }
            sh "docker build -t ${IMAGE_NAME} ."
        }
    }

    stage('Deploy Container') {
        steps {
            sh "docker stop ${IMAGE_NAME} || true"
            sh "docker rm ${IMAGE_NAME} || true"
            sh "docker run -d -p ${APP_PORT}:8080 --name ${IMAGE_NAME} ${IMAGE_NAME}"
        }
    }
}
}
