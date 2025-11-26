pipeline {
    agent any

    tools {
        maven 'maven'      // Your Maven installation name in Jenkins
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

        stage('Maven Build') {
            steps {
                sh '''
                    echo "Current directory:"
                    pwd

                    echo "Listing files before build:"
                    ls -R

                    mvn clean package -DskipTests

                    echo "Listing files after build:"
                    ls -R
                '''
            }
            post {
                success { echo "✅ Build successful" }
                failure { echo "❌ Build failed" }
                always  { echo "Maven Build completed" }
            }
        }

        stage('Detect JAR Name') {
            steps {
                script {
                    echo "🔍 Searching for JAR file (excluding *original* JARs)..."

                    // Search up to 4 levels deep for any .jar that is not *original*
                    env.APP_JAR = sh(
                        script: "find . -maxdepth 4 -type f -name '*.jar' ! -name '*original*' | head -n 1",
                        returnStdout: true
                    ).trim()

                    if (!env.APP_JAR) {
                        sh 'echo "Workspace tree for debugging:"; pwd; ls -R'
                        error("❌ No JAR file found anywhere in workspace (checked with find)")
                    }

                    echo "✅ Detected JAR File: ${env.APP_JAR}"
                }
            }
            post {
                success { echo "✅ JAR detection successful" }
                failure { echo "❌ JAR detection failed" }
                always  { echo "JAR detection stage completed" }
            }
        }

        stage('Run JAR with Conditions') {
            steps {
                script {
                    echo "Checking for existing running application..."

                    // Stop any previous app.jar started by Jenkins
                    sh "pkill -f app.jar || true"

                    echo "Removing old app.jar (if exists)"
                    sh "rm -f app.jar"

                    echo "Copying new JAR (${env.APP_JAR}) to app.jar"
                    sh "cp ${env.APP_JAR} app.jar"

                    echo "Starting Spring Boot on port 8088..."
                    sh "nohup java -jar app.jar --server.port=8088 > app.log 2>&1 &"

                    echo "Waiting 12 seconds for the app to start..."
                    sleep 12

                    def pid = sh(
                        script: "pgrep -f app.jar || true",
                        returnStdout: true
                    ).trim()

                    if (pid) {
                        echo "✅ Application started successfully with PID ${pid}"
                    } else {
                        echo "❌ Application failed to start, printing app.log:"
                        sh "cat app.log || true"
                        error("Application failed to start, see app.log above")
                    }
                }
            }
            post {
                success { echo "✅ Application started successfully" }
                failure { echo "❌ Application failed to start" }
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
