pipeline {
    agent any

    environment {
        APP_PORT = '9090'
        MAVEN_OPTS = '-Xms512m -Xmx1024m'
        JOB_NAME = ''
    }

    stages {
        stage('Initialize') {
            steps {
                script {
                    env.JOB_NAME = env.JOB_NAME
                    echo "Job Name: ${env.JOB_NAME}"
                }
            }
        }
        stage('Build') {
            steps {
                script {
                    try {
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
                                        sh 'java -jar contact.war'
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
                    steps {
                        sleep time: 30, unit: 'SECONDS'
                        echo "Running RestIT integration test..."
                        echo "Using Global Job Name: ${env.JOB_NAME}"
                        sh 'mvn -B -Dtest=RestIT -DskipITs=false test'
                    }
                }
            }
        }
    }
}
