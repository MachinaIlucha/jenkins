pipeline {
    agent any

    environment {
        APP_PORT = '9090'
        MAVEN_OPTS = '-Xms512m -Xmx1024m'
    }

    def jobName

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
                    timeout(time: 60, unit: 'SECONDS') {
                        steps {
                            script {
                                try {
                                    dir("target") {
                                        echo "Starting contact.war application on port ${APP_PORT}..."
                                        sh 'java -jar contact.war'
                                    }
                                    echo "Application started successfully."
                                } catch (Exception e) {
                                    echo "Task failed or timed out: ${e}"
                                }
                            }
                        }
                    }
                }
                stage('Running Test') {
                    steps {
                        sleep time: 30, unit: 'SECONDS'
                        echo "Running RestIT integration test..."
                        sh 'mvn -B -Dtest=RestIT -DskipITs=false test'
                    }
                }
            }
        }
    }
}
