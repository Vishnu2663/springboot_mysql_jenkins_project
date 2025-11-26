pipeline {
    agent any

    tools {
        maven 'maven'      // Your Maven name in Jenkins
    }

    environment {
        GIT_URL = "https://github.com/Vishnu2663/springboot_mysql_jenkins_project.git"
        CREDS = "git-credentials-id"
        BUILD_JAR = ""
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: "${GIT_URL}",
                    credentialsId: "${CREDS}"
            }
            post {
                success { echo "Checkout successful" }
                failure { echo "Checkout failed" }
                always  { echo "Checkout stage completed" }
            }
        }

        stage('Maven Build') {
            steps {
                sh "mvn clean package -DskipTests"
            }
            post {
                success { echo "Build successful" }
                failure { echo "Build failed" }
                always  { echo "Maven Build completed" }
            }
        }

        stage('Detect JAR Name') {
            steps {
                script {
                    BUILD_JAR = sh(
                        script: "ls target/*.jar | grep -v 'original' | head -n 1",
                        returnStdout: true
                    ).trim()

                    if (!BUILD_JAR) {
                        error("❌ No JAR file found in target/ directory")
                    }

                    echo "Detected JAR File: ${BUILD_JAR}"
                }
            }
            post {
                success { echo "JAR detection successful" }
                failure { echo "JAR detection failed" }
                always  { echo "JAR detection stage completed" }
            }
        }

        stage('Run JAR with Conditions') {
            steps {
                script {

                    echo "Checking for existing running application..."

                    def pid = sh(
                        script: "pgrep -f app.jar || true",
                        returnStdout: true
                    ).trim()

                    if (pid) {
                        echo "⚠ Old application running with PID ${pid}. Stopping..."
                        sh "kill -9 ${pid}"
                    } else {
                        echo "No running instance found."
                    }

                    echo "Removing old app.jar (if exists)"
                    sh "rm -f app.jar"

                    echo "Copying new JAR to app.jar"
                    sh "cp ${BUILD_JAR} app.jar"

                    echo "Starting Spring Boot on port 8088..."
                    sh "nohup java -jar app.jar --server.port=8088 > app.log 2>&1 &"
                }
            }
            post {
                success { echo "Application started successfully" }
                failure { echo "Application failed to start" }
                always  { echo "Run JAR stage completed" }
            }
        }
    }

    post {
        success { echo "🎉 Pipeline completed successfully!" }
        failure { echo "❌ Pipeline failed!" }
        always  { echo "🏁 Pipeline ended." }
    }
}
