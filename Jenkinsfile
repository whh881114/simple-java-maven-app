pipeline {
  agent { label 'maven-jdk17' } // 你在 Kubernetes 中部署的 Maven 构建 agent 标签

  environment {
    APP_NAME = "simple-java-maven-app"
    JOB_NAME = "${env.JOB_NAME}"
    GIT_COMMIT = "${env.JOB_NAME}"
    WORKSPACE = "${env.WORKSPACE}"
    REGISTRY = "harbor.idc.roywong.work"    // 内部docker仓库地址
    REPOSITORY = "library"                  // 构建后的镜像存放在内部docker仓库中的哪个项目中
    DOCKERFILE = "Dockerfile"
    IMAGE = "${env.REGISTRY}/${env.REPOSITORY}/${env.APP_NAME}:${GIT_COMMIT}"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Print Variables') {
      steps {
        echo "APP_NAME = ${env.APP_NAME}"
        echo "JOB_NAME = ${env.JOB_NAME}"
        echo "GIT_COMMIT = ${env.GIT_COMMIT}"
        echo "WORKSPACE = ${env.WORKSPACE}"
        echo "REGISTRY = ${env.REGISTRY}"
        echo "REPOSITORY = ${env.REPOSITORY}"
        echo "DOCKERFILE = ${env.DOCKERFILE}"
        echo "IMAGE = ${env.IMAGE}"
      }
    }

    stage('Build') {
      steps {
        sh 'mvn clean compile'
      }
    }

    stage('Package') {
      steps {
        sh 'mvn package -DskipTests'
        archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
      }
    }

    stage('Trigger Docker Job') {
      steps {
        build job: 'pipeline-idc-registry', wait: false, parameters: [
          string(name: 'APP_NAME', value: "${env.APP_NAME}"),
          string(name: 'JOB_NAME', value: "${env.JOB_NAME}"),
          string(name: 'GIT_COMMIT', value: "${env.GIT_COMMIT}"),
          string(name: 'REGISTRY', value: "${env.REGISTRY}"),
          string(name: 'REPOSITORY', value: "${env.REPOSITORY}"),
          string(name: 'IMAGE', value: "${env.IMAGE}"),
          string(name: 'WORKSPACE', value: "${env.WORKSPACE}"),
          string(name: 'DOCKERFILE', value: "${env.DOCKERFILE}"),
        ]
      }
    }
  }
}
