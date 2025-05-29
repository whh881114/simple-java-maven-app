pipeline {
  agent { label 'maven-jdk17' } // 你在 Kubernetes 中部署的 Maven 构建 agent 标签

  environment {
    APP_NAME = "simple-java-maven-app"
    JOB_NAME = "${env.JOB_NAME}"
    GIT_COMMIT = ""                         // 稍后动态赋值
    WORKSPACE = "${env.WORKSPACE}"
    REGISTRY = "harbor.idc.roywong.work"    // 内部docker仓库地址
    REPOSITORY = "library"                  // 构建后的镜像存放在内部docker仓库中的哪个项目中
    DOCKERFILE = "Dockerfile"
    IMAGE = ""                              // 稍后动态赋值
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Set Variables') {
      steps {
        script {
          env.GIT_COMMIT = sh(script: "git rev-parse HEAD", returnStdout: true).trim()
          env.IMAGE = "${env.REGISTRY}/${env.REPOSITORY}/${env.APP_NAME}:${env.GIT_COMMIT}"
        }
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
