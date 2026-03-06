pipeline {
    agent { label 'maven-agent' }

    environment {
        APP_HOST = '10.0.1.177'   // replace with your application server private IP
        APP_USER = 'ubuntu'
        REMOTE_JAR = '/home/ubuntu/app.jar'
        LOG_FILE = '/home/ubuntu/app.log'
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build & Test') {
            steps {
                sh 'mvn clean test'
            }
        }

        stage('Security Scan') {
            steps {
                sh 'trivy fs --exit-code 1 --severity HIGH,CRITICAL .'
            }
        }

        stage('Package') {
            steps {
                sh 'mvn package'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }

        stage('Deploy to Application Server') {
            when {
                branch 'main'
            }
            steps {
                sshagent(['application-server']) {
                    sh '''
                    JAR=$(ls -1 target/*.jar | head -n 1)

                    scp -o StrictHostKeyChecking=no "$JAR" ${APP_USER}@${APP_HOST}:${REMOTE_JAR}

                    ssh -o StrictHostKeyChecking=no ${APP_USER}@${APP_HOST} "
                        pkill -f ${REMOTE_JAR} || true
                        nohup java -jar ${REMOTE_JAR} > ${LOG_FILE} 2>&1 &
                    "
                    '''
                }
            }
        }
    }
}
