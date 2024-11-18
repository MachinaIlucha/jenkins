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
                echo "Listing files in target directory:"
                sh 'ls -l target/'
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
                                    def warFile = sh(script: 'ls *.war', returnStdout: true).trim()
                                    if (warFile) {
                                        echo "Starting the application with WAR file: ${warFile}"
                                        sh "java -jar ${warFile} --server.port=${APP_PORT} &"
                                    } else {
                                        error "WAR file not found in target directory."
                                    }
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

