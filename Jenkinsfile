def jobName = ''

pipeline {
    agent any

    environment {
        APP_PORT = '9090'
        MVN_GOALS = 'clean package'
        WAR_FILE = 'contact.war'
    }

    stages {
        stage('Initialize') {
            steps {
                script {
                    jobName = env.JOB_NAME
                    echo "Job Name: ${jobName}"
                }
            }
        }

        stage('Build') {
            steps {
                echo "Building the project..."

                sh "mvn ${MVN_GOALS}"
            }
        }

        stage('Integration Test') {
            parallel {
                stage('Running Application') {
                    agent any
                    steps {
                        script {
                            echo "Launching application on port ${APP_PORT}..."
                            try {
                                timeout(time: 60, unit: 'SECONDS') {
                                    dir('target') {
                                        sh "java -jar ${WAR_FILE} --server.port=${APP_PORT}"
                                    }
                                }
                            } catch (Exception e) {
                                echo "Application timed out after 60 seconds: ${e}"
                                currentBuild.result = 'SUCCESS'
                            }
                        }
                    }
                }
                stage('Running Test') {
                    steps {
                        script {
                            try {
                                echo "Waiting for the application to start..."
                                sleep(time: 30, unit: 'SECONDS')
                                echo "Running RestIT integration tests..."

                                sh 'mvn test -Dtest=RestIT'
                            } catch (Exception e) {
                                echo "Tests failed: ${e}"
                                error("Test execution failed")
                            }
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Build, application run, and tests completed successfully!"
        }
        failure {
            echo "Build or tests encountered a failure."
        }
        unstable {
            echo "Build completed but marked as unstable."
        }
        changed {
            echo "Build status has changed from the previous run."
        }
    }
}

