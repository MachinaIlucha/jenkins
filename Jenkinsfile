pipeline {
    agent any

    environment {
        APP_PORT = '9090'
        MAVEN_OPTS = '-Xms512m -Xmx1024m'
        JOB_NAME = "${env.JOB_NAME}"
    }

    stages {
        stage('Build') {
            steps {
                script {
                    try {
                        echo "Building the project in job: ${JOB_NAME}"
                        sh 'mvn clean -B package -DskipTests'
                    } catch (Exception e) {
                        echo "Build failed: ${e.message}"
                        currentBuild.result = 'FAILURE'
                        error("Stopping the pipeline due to build failure")
                    }
                }
            }
        }

        stage('Integration Test') {
            parallel {
                stage('Running Application') {
                    agent any
                    steps {
                        timeout(time: 60, unit: 'SECONDS') {
                            script {
                                try {
                                    dir("target") {
                                        echo "Starting contact.war application on port ${APP_PORT}..."
                                        sh 'nohup java -jar contact.war &'
                                    }
                                    echo "Application started successfully."
                                } catch (Exception e) {
                                    echo "Task failed or timed out: ${e.message}"
                                }
                            }
                        }
                    }
                }
                stage('Running Test') {
                    agent any
                    steps {
                        script {
                            echo "Waiting for the application to start"
                            sleep(time: 30, unit: 'SECONDS')
                        }
                        echo "Running RestIT integration test..."
                        echo "Using Global Job Name: ${JOB_NAME}"
                        sh 'mvn -B -Dtest=RestIT test'
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Build and tests completed successfully for ${JOB_NAME}"
        }
        failure {
            echo "Build or tests failed for ${JOB_NAME}"
        }
    }
}
