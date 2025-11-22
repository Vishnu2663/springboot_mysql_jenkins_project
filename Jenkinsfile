pipeline {
    agent any

    environment {
        APP_NAME    = "myapp"
        APP_SERVER  = "172.31.17.196"          // change this
        SSH_CRED_ID = "app-server-ssh"
        DEPLOY_DIR  = "/opt/myapp"
        JAR_NAME    = "app.jar"
        GIT_BRANCH  = "main"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: "${GIT_BRANCH}", url: 'https://github.com/Vishnu2663/springboot_cicd_jenkins_project.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Copy Artifact to App Server') {
            steps {
                sshagent (credentials: [env.SSH_CRED_ID]) {
                    sh """
                        JAR_FILE=\$(ls target/*.jar | head -n 1)
                        echo "Using jar: \$JAR_FILE"

                        scp -o StrictHostKeyChecking=no "\$JAR_FILE" deploy@${APP_SERVER}:${DEPLOY_DIR}/${JAR_NAME}
                    """
                }
            }
        }

        stage('Restart Service on App Server') {
            steps {
                sshagent (credentials: [env.SSH_CRED_ID]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no deploy@${APP_SERVER} \\
                          'sudo systemctl restart ${APP_NAME}.service && sudo systemctl status ${APP_NAME}.service --no-pager'
                    """
                }
            }
        }

     stage('Health Check') {
    steps {
        script {
            sh """
                echo "Health Checking..."
                sleep 5
                curl -s http://${APP_SERVER}:8080/actuator/health > /dev/null
            """
        }
    }
}

        }
    }

    post {
        success {
            echo "Deployment success! App should be running on http://${APP_SERVER}:8080/"
        }
        failure {
            echo "Deployment failed. Check console output."
        }
    }
}
