pipeline {
  agent { label 'maven-jdk17' } // 你在 Kubernetes 中部署的 Maven 构建 agent 标签

  environment {
    APP_NAME = "simple-java-maven-app"
    GIT_COMMIT = "${env.GIT_COMMIT}"
    WORKSPACE = "${env.WORKSPACE}"
    REGISTRY = "harbor.idc.roywong.work"    // 内部docker仓库地址
    REPOSITORY = "library"                  // 构建后的镜像存放在内部docker仓库中的哪个项目中
    DOCKERFILE = "Dockerfile"
    IMAGE = "${env.REGISTRY}/${env.REPOSITORY}/${env.APP_NAME}:${GIT_COMMIT}"

    // ARGOCD变量，ARGOCD_URL为内网地址，ARGOCD_TOKEN仅拥有刷新和同步权限。
    ARGOCD_URL = "https://argocd-server.idc-ingress-nginx-lan.roywong.work:443"
    ARGOCD_TOKEN = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJhcmdvY2QiLCJzdWIiOiJyZWZyZXNoLXN5bmMtdXNlcjphcGlLZXkiLCJuYmYiOjE3NDc4MDk5NjIsImlhdCI6MTc0NzgwOTk2MiwianRpIjoiNGI3ODljMzQtODdhZi00YzIxLWE2YmYtMDA0ZThhNzY5ZDBiIn0.Q4krUNzorTBqdDSIp2jR6eqDp-a2IESRxvzVZuopJfA"
    ARGOCD_APP = $APP_NAME
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
        build job: 'pipeline-idc-registry', wait: true, parameters: [
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

    stage('Trigger Argocd Job') {
      steps {
        build job: 'pipeline-sync-argocd-app', wait: true, parameters: [
          string(name: 'ARGOCD_URL', value: "${env.ARGOCD_URL}"),
          string(name: 'ARGOCD_TOKEN', value: "${env.ARGOCD_TOKEN}"),
          string(name: 'ARGOCD_APP', value: "${env.ARGOCD_APP}"),
        ]
      }
    }
  }
}
