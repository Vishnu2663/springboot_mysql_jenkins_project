pipeline {
    agent any

    tools {
        maven 'maven'
    }

    stages {

        stage('Build') {
            steps {
                echo "Running Maven Build..."
                sh 'mvn clean package -DskipTests'
            }
            post {
                success { echo "Build SUCCESS ✔" }
                failure { echo "Build FAILED ❌" }
                always  { echo "Build FINISHED" }
            }
        }

        stage('Deploy') {
            steps {
                echo "Deploying Spring Boot App on 8088..."
                sh '''#!/bin/bash
                set -e

                JAR="spring_app_sak-0.0.1-SNAPSHOT.jar"
                PATH_JAR="target/$JAR"
                LOG="/tmp/app8088.log"

                echo "Removing old log..."
                rm -f "$LOG"

                echo "Stopping old running app..."
                if pgrep -f "$JAR" > /dev/null; then
                    echo "Old app found → stopping..."
                    pkill -f "$JAR"
                    sleep 5
                else
                    echo "No old app running."
                fi

                echo "Starting new app..."
                nohup java -jar "$PATH_JAR" > "$LOG" 2>&1 &

                echo "Waiting 15 seconds..."
                sleep 15

                if pgrep -f "$JAR" > /dev/null; then
                    echo "✔ Application started successfully!"
                else
                    echo "❌ Application failed to start!"
                    echo "===== APP LOG ====="
                    cat "$LOG"
                    echo "===================="
                    exit 1
                fi
                '''
            }
            post {
                success { echo "Deploy SUCCESS ✔" }
                failure { echo "Deploy FAILED ❌" }
                always  { echo "Deploy FINISHED" }
            }
        }
    }

    post {
        success { echo "PIPELINE SUCCESS ✔" }
        failure { echo "PIPELINE FAILED ❌" }
        always  { echo "PIPELINE FINISHED" }
    }
}
