// Jenkinsfile (Declarative)
pipeline {
  agent any

  environment {
    APP_NAME = "myapp"
    DEPLOY_USER = "deploy"
    DEPLOY_HOST = "APP_IP"        // <-- replace
    REMOTE_APP_DIR = "/opt/apps/${APP_NAME}"
    SSH_CREDS = "deploy-app-ssh-key" // Jenkins credential ID (from step above)
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Build') {
      steps {
        sh 'mvn -B clean package -DskipTests=false'
      }
      post {
        success { archiveArtifacts artifacts: "target/*.jar", fingerprint: true }
      }
    }

    stage('Deploy') {
      steps {
        sshagent (credentials: [env.SSH_CREDS]) {
          script {
            def jar = sh(script: "ls target/*.jar | head -n1", returnStdout: true).trim()
            if (!jar) { error "Jar not found" }

            // unique release dir
            def ts = sh(script: "date +%Y%m%d%H%M%S", returnStdout: true).trim()
            def release = "release-${BUILD_ID}-${ts}"

            // copy jar to remote temp location (home directory)
            sh "scp -o StrictHostKeyChecking=no ${jar} ${DEPLOY_USER}@${DEPLOY_HOST}:~/myapp.jar"

            // remote commands: move jar to release dir, update symlink, restart
            sh """
            ssh -o StrictHostKeyChecking=no ${DEPLOY_USER}@${DEPLOY_HOST} \\
            'mkdir -p ${REMOTE_APP_DIR}/releases/${release} && \\
             mv ~/myapp.jar ${REMOTE_APP_DIR}/releases/${release}/myapp.jar && \\
             ln -sfn ${REMOTE_APP_DIR}/releases/${release} ${REMOTE_APP_DIR}/current && \\
             sudo /bin/systemctl restart ${APP_NAME}.service'
            """
          }
        }
      }
    }

    stage('Smoke Test') {
      steps {
        echo "Waiting 5s then checking app health..."
        sleep 5
        sh "curl -sS http://${DEPLOY_HOST}:8080/actuator/health || true"
      }
    }
  }

  post {
    success { echo "Deployed successfully: ${env.BUILD_URL}" }
    failure { echo "Deployment failed, check logs." }
  }
}
