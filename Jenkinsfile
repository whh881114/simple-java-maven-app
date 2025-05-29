pipeline {
  agent { label 'maven-jdk17' } // 你在 Kubernetes 中部署的 Maven 构建 agent 标签

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build') {
      steps {
        sh 'mvn clean compile'
      }
    }

    stage('Test') {
      steps {
        sh 'mvn test'
      }
    }

    stage('Package') {
      steps {
        sh 'mvn package -DskipTests'
        archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
      }
    }
  }
}
