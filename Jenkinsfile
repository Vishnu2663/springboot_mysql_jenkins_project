pipeline {
    agent any

    tools {
        maven 'maven'
    }

    environment {
        GIT_URL = "https://github.com/Vishnu2663/springboot_mysql_jenkins_project.git"
        CREDS   = "git-credentials-id"
        APP_JAR = ""
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: "${GIT_URL}",
                    credentialsId: "${CREDS}"
            }
            post {
                success { echo "✅ Checkout successful" }
                failure { echo "❌ Checkout failed" }
                always  { echo "Checkout stage completed" }
            }
        }

        stage('Build') {
            steps {
                echo "Running Maven build..."
                sh '''
                    mvn clean package -DskipTests
                    echo "Listing target directory:"
                    ls -lh target/
                '''
            }
            post {
                success { echo "✅ Build successful" }
                failure { echo "❌ Build failed" }
                always  { echo "Build stage completed" }
            }
        }

        stage('Detect JAR') {
            steps {
                script {
                    env.APP_JAR = sh(
                        script: "ls target/*.jar | grep -v original | head -n 1",
                        returnStdout: true
                    ).trim()

                    if (!env.APP_JAR) {
                        error("❌ No JAR file found in target directory!")
                    }

                    echo "✅ Detected JAR: ${env.APP_JAR}"
                }
            }
            post {
                success { echo "✅ JAR detection successful" }
                failure { echo "❌ JAR detection failed" }
                always  { echo "Detect JAR stage completed" }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    echo "Checking for existing running app..."

                    def pid = sh(
                        script: "pgrep -f app.jar || true",
                        returnStdout: true
                    ).trim()

                    if (pid) {
                        echo "⚠ Stopping old app with PID: ${pid}"
                        sh "kill -9 ${pid}"
                        sleep 5
                    } else {
                        echo "✅ No running instance found."
                    }

                    echo "Removing old app.jar"
                    sh "rm -f app.jar"

                    echo "Copying new JAR as app.jar"
                    sh "cp ${env.APP_JAR} app.jar"

                    echo "Starting Spring Boot on port 8088..."
                    sh """
                        nohup java -jar app.jar --server.port=8088 > app.log 2>&1 &
                    """

                    sleep 12

                    def newPid = sh(
                        script: "pgrep -f app.jar || true",
                        returnStdout: true
                    ).trim()

                    if (!newPid) {
                        echo "======== APPLICATION LOG ========"
                        sh "cat app.log"
                        error("❌ Application failed to start!")
                    } else {
                        echo "✅ Application started successfully with PID ${newPid}"
                    }
                }
            }
            post {
                success { echo "✅ Deploy successful" }
                failure { echo "❌ Deploy failed" }
                always  { echo "Deploy stage completed" }
            }
        }
    }

    post {
        success { echo "🎉 PIPELINE COMPLETED SUCCESSFULLY" }
        failure { echo "❌ PIPELINE FAILED" }
        always  { echo "🏁 PIPELINE FINISHED" }
    }
}
