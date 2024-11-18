pipeline {
    agent any
    environment {
        APP_PORT = '9090'
        JOB_NAME = env.JOB_NAME
    }
    stages {
        stage('Build') {
            steps {
                echo "Building the project..."
                sh 'mvn clean package'
            }
        }

        stage('Integration Test') {
            parallel {
                stage('Running Application') {
                    agent any
                    options {
                        timeout(time: 60, unit: 'SECONDS')
                    }
                    steps {
                        script {
                            try {
                                dir('target') {
                                    echo "Launching contact.war on port ${APP_PORT}..."
                                    sh 'nohup java -jar contact.war &'
                                }
                            } catch (Exception e) {
                                echo "Application launch timed out. Exiting stage."
                                currentBuild.result = 'SUCCESS'
                            }
                        }
                    }
                }

                stage('Running Test') {
                    steps {
                        echo "Waiting for the application to start..."
                        sleep 30
                        echo "Running RestIT integration test..."
                        sh 'mvn -Dtest=RestIT test'
                    }
                }
            }
        }
    }
    post {
        always {
            echo "Cleaning up background processes..."
            sh 'pkill -f contact.war || true'
        }
        success {
            echo "Build and integration test completed successfully."
        }
        failure {
            echo "Build or integration test failed."
        }
    }
}

