pipeline {
    agent any

    environment {
        APP_PORT = "8081"
        APP_JAR  = "target/spring-petclinic-*.jar"
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/spring-projects/spring-petclinic.git',
                    branch: 'main'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
            post {
                success { echo 'Build succeeded.' }
                failure { error 'Build failed — check Maven output above.' }
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit testResults: 'target/surefire-reports/*.xml',
                          allowEmptyResults: true
                }
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    # Stop any previous instance
                    pkill -f "spring-petclinic" || true
                    sleep 2

                    # Start the application in the background
                    nohup java -jar ${APP_JAR} \
                        --server.port=${APP_PORT} \
                        > /tmp/petclinic.log 2>&1 &

                    echo "Waiting for application to start..."
                    sleep 20
                '''
            }
        }

        stage('Verify') {
            steps {
                sh '''
                    echo "Checking application health..."
                    curl --fail --silent --max-time 10 http://localhost:${APP_PORT}/actuator/health \
                        || curl --fail --silent --max-time 10 http://localhost:${APP_PORT}/
                    echo "Application is running at http://localhost:${APP_PORT}"
                '''
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully. Access the app at http://localhost:${APP_PORT}"
        }
        failure {
            echo "Pipeline failed. Check the logs above for details."
            sh 'cat /tmp/petclinic.log || true'
        }
    }
}
