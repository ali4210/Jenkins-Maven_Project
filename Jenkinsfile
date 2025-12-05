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

/*
pipeline { agent any

parameters {
    choice(name: 'ACTION', choices: ['Deploy', 'Destroy'], description: 'Choose whether to Deploy or Destroy the app')
}

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
        when {
            expression { params.ACTION == 'Deploy' }
        }
        steps {
            echo 'Action is Deploy: Compiling Code...'
            sh 'mvn clean package'
        }
    }

    stage('Build Docker Image') {
        when {
            expression { params.ACTION == 'Deploy' }
        }
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
        when {
            expression { params.ACTION == 'Deploy' }
        }
        steps {
            echo 'Deploying to Server...'
            sh "docker stop ${IMAGE_NAME} || true"
            sh "docker rm ${IMAGE_NAME} || true"
            sh "docker run -d -p ${APP_PORT}:8080 --name ${IMAGE_NAME} ${IMAGE_NAME}"
        }
    }

    stage('Destroy Container') {
        when {
            expression { params.ACTION == 'Destroy' }
        }
        steps {
            echo 'Destroying Environment...'
            sh "docker stop ${IMAGE_NAME} || true"
            sh "docker rm ${IMAGE_NAME} || true"
            echo "Container ${IMAGE_NAME} has been removed."
        }
    }
}
}

*/


// Modified Version with SonarQube Scanner embedded

/*
pipeline { agent any

parameters {
    choice(name: 'ACTION', choices: ['Deploy', 'Destroy'], description: 'Choose whether to Deploy or Destroy the app')
}

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
        when {
            expression { params.ACTION == 'Deploy' }
        }
        steps {
            echo 'Compiling and Building JAR...'
            sh 'mvn clean package'
        }
    }

    stage('SonarQube Analysis') {
        when {
            expression { params.ACTION == 'Deploy' }
        }
        steps {
            script {
                // This pulls the tool you named 'SonarScanner' in Jenkins configuration
                def scannerHome = tool 'SonarScanner'
                
                // Connects to the server you named 'sonar-server' in Jenkins configuration
                withSonarQubeEnv('sonar-server') {
                    sh """
                        ${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=my-java-app \
                        -Dsonar.sources=src \
                        -Dsonar.java.binaries=target/classes
                    """
                }
            }
        }
    }

    stage('Build Docker Image') {
        when {
            expression { params.ACTION == 'Deploy' }
        }
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
        when {
            expression { params.ACTION == 'Deploy' }
        }
        steps {
            echo 'Deploying to Server...'
            sh "docker stop ${IMAGE_NAME} || true"
            sh "docker rm ${IMAGE_NAME} || true"
            sh "docker run -d -p ${APP_PORT}:8080 --name ${IMAGE_NAME} ${IMAGE_NAME}"
        }
    }

    stage('Destroy Container') {
        when {
            expression { params.ACTION == 'Destroy' }
        }
        steps {
            echo 'Destroying Environment...'
            sh "docker stop ${IMAGE_NAME} || true"
            sh "docker rm ${IMAGE_NAME} || true"
            echo "Container ${IMAGE_NAME} has been removed."
        }
    }
}
}
*/

pipeline { agent any

parameters {
    choice(name: 'ACTION', choices: ['Deploy', 'Destroy'], description: 'Choose whether to Deploy or Destroy the app')
}

tools {
    maven 'MyLocalMaven'
    jdk 'JDK-17'
    // We still need the tool definition to find the scanner binary
    sonarQubeScanner 'SonarScanner'
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
        when {
            expression { params.ACTION == 'Deploy' }
        }
        steps {
            echo 'Compiling and Building JAR...'
            sh 'mvn clean package'
        }
    }

    stage('SonarQube Analysis') {
        when {
            expression { params.ACTION == 'Deploy' }
        }
        steps {
            script {
                // 1. Get the path where Jenkins installed the scanner
                def scannerHome = tool 'SonarScanner'
                
                // 2. Run the scan using the token DIRECTLY (Bypassing Jenkins Credentials)
                // REPLACE THE TEXT BELOW WITH YOUR ACTUAL TOKEN
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
        when {
            expression { params.ACTION == 'Deploy' }
        }
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
        when {
            expression { params.ACTION == 'Deploy' }
        }
        steps {
            echo 'Deploying to Server...'
            sh "docker stop ${IMAGE_NAME} || true"
            sh "docker rm ${IMAGE_NAME} || true"
            sh "docker run -d -p ${APP_PORT}:8080 --name ${IMAGE_NAME} ${IMAGE_NAME}"
        }
    }

    stage('Destroy Container') {
        when {
            expression { params.ACTION == 'Destroy' }
        }
        steps {
            echo 'Destroying Environment...'
            sh "docker stop ${IMAGE_NAME} || true"
            sh "docker rm ${IMAGE_NAME} || true"
            echo "Container ${IMAGE_NAME} has been removed."
        }
    }
}
}
