pipeline { agent any

parameters {
    choice(name: 'ACTION', choices: ['Deploy', 'Destroy'], description: 'Choose Action')
}

tools {
    maven 'MyLocalMaven'
    jdk 'JDK-17'
    // I REMOVED the sonar line here to fix your syntax error
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
                // FIX: We point directly to the folder where you installed it
                def scannerHome = '/opt/sonar-scanner'
                
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
