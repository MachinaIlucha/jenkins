def jobName = ''

pipeline {
    agent any
    environment {
        APP_PORT = '9090'
    }
    stages {
        stage('Initialize') {
            steps {
                script {
                    jobName = env.JOB_NAME
                }
            }
        }
        stage('Build') {
            steps {
                sh 'mvn package'
            }
        }
        stage('Integration Test') {
            parallel {
                stage('Running Application') {
                    agent any
                    steps {
                        script {
                            try {
                                timeout(time: 60, unit: 'SECONDS') {
                                    dir('target') {
                                        sh 'java -jar contact.war'
                                    }
                                }
                            } catch (err) {
                                echo 'Application stopped after 60 seconds'
                                currentBuild.result = 'SUCCESS'
                            }
                        }
                    }
                }
                stage('Running Test') {
                    steps {
                        sleep 30
                        sh 'mvn test -Dtest=RestIT'
                    }
                }
            }
        }
    }
}

