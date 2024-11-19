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
                    echo "🔧 Initialization started."
                    echo "📝 Job Name: ${jobName}"
                    echo "🌐 Application will run on port: ${APP_PORT}"
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    try {
                        echo "🛠️ Starting build process."
                        echo "⚙️ Running Maven goals: ${MVN_GOALS}"

                        sh """
                            set -e
                            mvn ${MVN_GOALS}
                        """
                        echo "✅ Build process completed successfully."
                    } catch (Exception e) {
                        echo "❌ Build failed: ${e.message}"
                        error("❗ Build stage encountered an error.")
                    }
                }
            }
        }

        stage('Integration Test') {
            parallel {
                stage('Running Application') {
                    agent any
                    steps {
                        script {
                            try {
                                echo "🚀 Preparing to launch the application."
                                echo "📦 WAR file to be used: ${WAR_FILE}"

                                timeout(time: 60, unit: 'SECONDS') {
                                    dir('${env.WORKSPACE}/target') {
                                        echo "🌐 Launching the application on port ${APP_PORT}."
                                        sh """
                                            set -e
                                            java -jar ${WAR_FILE}
                                        """
                                        echo "✅ Application is running."
                                    }
                                }
                            } catch (Exception e) {
                                echo "⏳ Application timed out after 60 seconds: ${e.message}"
                                echo "⚠️ Proceeding despite the application timeout."
                                currentBuild.result = 'SUCCESS'
                            }
                        }
                    }
                }

                stage('Running Test') {
                    steps {
                        script {
                            try {
                                echo "⏳ Waiting for the application to be ready."
                                sleep(time: 30, unit: 'SECONDS')

                                echo "🧪 Starting RestIT integration tests."
                                sh """
                                    set -e
                                    mvn test -Dtest=RestIT
                                """
                                echo "✅ RestIT integration tests completed."
                            } catch (Exception e) {
                                echo "❌ Test execution failed: ${e.message}"
                                error("❗ Test execution encountered an error.")
                            }
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            echo "🎉 Build, application run, and tests completed successfully."
        }
        failure {
            echo "💥 Build or tests failed."
        }
        unstable {
            echo "⚠️ Build completed but marked as unstable."
        }
        changed {
            echo "🔄 Build status has changed compared to the previous run."
        }
    }
}
