pipeline {
  agent { label 'maven-agent' }

  stages {
    stage('Checkout Code') {
      steps {
        checkout scm
      }
    }

    stage('Build & Test') {
      steps {
        sh 'mvn -B clean test'
      }
    }

    stage('Security Scan') {
      steps {
        sh 'trivy fs --exit-code 1 --severity HIGH,CRITICAL .'
      }
    }

    stage('Package') {
      steps {
        sh 'mvn -B package'
        archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
      }
    }

    stage('Deploy to Application Server') {
      steps {
        sshagent(['application-server']) {
          sh '''
            set -e
            JAR=$(ls -1 target/*.jar | head -n 1)

            scp -o StrictHostKeyChecking=no "$JAR" ubuntu@100.53.50.199:/home/ubuntu/app.jar

            ssh -o StrictHostKeyChecking=no ubuntu@100.53.50.199 \
              "pkill -f /home/ubuntu/app.jar || true; nohup java -jar /home/ubuntu/app.jar > /home/ubuntu/app.log 2>&1 &"
          '''
        }
      }
    }
  }
}
