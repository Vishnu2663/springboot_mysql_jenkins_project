pipeline {
    // 1. Define where the job runs
    agent any 
    
    // 2. Define environment variables
    environment {
        // Change this if your JAR name changes
        APP_NAME = 'demo-0.0.1-SNAPSHOT' 
        // This assumes you are using Maven and the JAR is in the default location
        JAR_PATH = "target/${APP_NAME}.jar" 
    }
    
    // 3. Define the steps (stages) in the CI process
    stages {
        
        stage('Checkout Source Code') {
            steps {
                // Ensure the workspace is clean before starting the build
                cleanWs() 
                // Jenkins automatically checks out the code from the SCM configured in the job
            }
        }
        
        stage('Build & Test') {
            steps {
                // Use the sh step to execute a shell command
                // 'clean' deletes the existing 'target' folder (important!)
                // 'package' runs tests and creates the JAR file
                sh 'mvn clean package -DskipTests'
                
                // You can add a separate stage for tests if needed:
                // sh 'mvn test' 
            }
        }
        
        stage('Archive Artifact') {
            steps {
                // Archive the generated JAR file from the 'target' folder
                // This makes the JAR available for download on the Jenkins job page
                archiveArtifacts artifacts: "${JAR_PATH}", fingerprint: true
            }
        }
        
        // This stage is optional, depending on how you deploy
        stage('Deploy') { 
            steps {
                // Example: Deploying the JAR to a remote server via SSH
                // NOTE: This requires SSH setup/plugins in Jenkins
                
                // You will need to replace the PLACEHOLDERS below
                sh "scp ${JAR_PATH} user@your-server-ip:/path/to/deploy/" 
                
                // After copying, you would typically run a command on the remote server to start the app
                // sh 'ssh user@your-server-ip "nohup java -jar /path/to/deploy/${JAR_PATH} &"'
            }
        }
    }
}
