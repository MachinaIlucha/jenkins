pipeline {
    agent any

    environment {
        APP_PORT = '9090'
        JOB_NAME = "${env.JOB_NAME}"
    }

    stages {
        stage('Build') {
            steps {
                script {
                    try {
                        echo "Building the project"
                        sh 'mvn clean package -B -DskipTests'

                        echo "Listing files in the target directory after build:"
                        sh 'ls -la target'

                        echo "Stashing contact.war file"
                        stash includes: 'target/contact.war', name: 'contact-war'
                    } catch (Exception e) {
                        echo "Build failed: ${e.message}"
                        currentBuild.result = 'FAILURE'
                        error("Build stage failed, stopping the pipeline.")
                    }
                }
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
                                echo "Unstashing contact.war file"
                                unstash 'contact-war'

                                dir("target") {
                                    echo "Starting contact.war application on port ${APP_PORT}..."
                                    sh 'pwd && ls -la && nohup java -jar ./contact.war > nohup.out 2>&1 &'
                                }
                            } catch (Exception e) {
                                echo "Application stage timed out or failed."
                                currentBuild.result = 'SUCCESS'
                            }
                        }
                    }
                }

                stage('Running Test') {
                    steps {
                        script {
                            echo "Waiting for the application to start..."
                            sleep(time: 30, unit: 'SECONDS')

                            echo "Running the RestIT integration tests"
                            sh 'mvn -Dtest=RestIT test -B'
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed. Check the logs for more details."
        }
    }
}

