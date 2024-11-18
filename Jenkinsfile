pipeline {
    agent any

    environment {
        APP_PORT = '9090'
        JOB_NAME = "${env.JOB_NAME}"
    }

    stages {
        stage('Build') {
            steps {
                echo "Building the project: ${JOB_NAME}"
                sh 'mvn clean package -DskipTests'
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
                                    echo "Starting the application on port ${APP_PORT}"
                                    sh 'java -jar contact.war --server.port=${APP_PORT} &'
                                }
                            } catch (Exception e) {
                                echo "Application run terminated due to timeout or error: ${e.getMessage()}"
                                currentBuild.result = 'SUCCESS'
                            }
                        }
                    }
                }

                stage('Running Test') {
                    steps {
                        echo "Waiting for the application to start..."
                        sleep 30

                        echo "Running integration tests (RestIT)"
                        sh 'mvn test -Dtest=RestIT'
                    }
                }
            }
        }
    }
}

