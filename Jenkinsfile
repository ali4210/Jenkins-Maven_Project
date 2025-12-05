pipeline {
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
                git 'https://github.com/jenkins-docs/simple-java-maven-app.git'
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
