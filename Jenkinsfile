pipeline {
  
  environment {
    PROJECT = "practice-357012"
    APP_NAME = "gceme"
    DOCKER_REPO = "arifpradana22"
    FE_SVC_NAME = "${APP_NAME}-frontend"
    CLUSTER = "jenkins-cd"
    CLUSTER_ZONE = "us-central1-c"
    IMAGE_TAG = "docker.io/${DOCKER_REPO}/${APP_NAME}:${env.BRANCH_NAME}.${env.BUILD_NUMBER}"
    JENKINS_CRED = "${PROJECT}"
    DOCKERHUB_CRED = credentials('dockerhub')
  }

  agent {
    kubernetes {
      // label 'sample-app'
      defaultContainer 'jnlp'
      yaml """
apiVersion: v1
kind: Pod
metadata:
labels:
  component: ci
spec:
  # Use service account that can deploy to all namespaces
  serviceAccountName: jenkins-sa
  containers:
  - name: golang
    image: golang:1.10
    command:
    - cat
    tty: true
  - name: docker
    image: docker:latest
    command:
    - cat
    tty: true
    volumeMounts:
    - mountPath: /var/run/docker.sock
      name: docker-sock
  volumes:
  - name: docker-sock
    hostPath:
      path: /var/run/docker.sock
  - name: kubectl
    image: gcr.io/cloud-builders/kubectl
    command:
    - cat
    tty: true
"""
}
  }
  stages {
    // stage('Test') {
    //   steps {
    //     container('golang') {
    //       sh """
    //         ln -s `pwd` /go/src/sample-app
    //         cd /go/src/sample-app
    //         go test
    //       """
    //     }
    //   }
    // }
    stage('Build image with docker') {
      steps {
        container('docker'){
        sh "docker build -t ${IMAGE_TAG} ."
        }
      }
    }
    stage('Login on Dockerhub') {
      steps {
        container('docker'){
        sh 'echo $DOCKERHUB_CRED_PSW | docker login -u $DOCKERHUB_CRED_USR --password-stdin'
        }
      }
    }
    stage('Push image with docker') {
      steps {
        container('docker'){
        sh "docker push ${IMAGE_TAG}"
        }
      }
    }
    // stage('Deploy Canary') {
    //   // Canary branch
    //   when { branch 'canary' }
    //   steps {
    //     container('kubectl') {
    //       // Change deployed image in canary to the one we just built
    //       sh("sed -i.bak 's#docker.io/arifpradana22/gceme:1.0.0#${IMAGE_TAG}#' ./k8s/canary/*.yaml")
    //       step([$class: 'KubernetesEngineBuilder', namespace:'production', projectId: env.PROJECT, clusterName: env.CLUSTER, zone: env.CLUSTER_ZONE, manifestPattern: 'k8s/services', credentialsId: env.JENKINS_CRED, verifyDeployments: false])
    //       step([$class: 'KubernetesEngineBuilder', namespace:'production', projectId: env.PROJECT, clusterName: env.CLUSTER, zone: env.CLUSTER_ZONE, manifestPattern: 'k8s/canary', credentialsId: env.JENKINS_CRED, verifyDeployments: true])
    //       sh("echo http://`kubectl --namespace=production get service/${FE_SVC_NAME} -o jsonpath='{.status.loadBalancer.ingress[0].ip}'` > ${FE_SVC_NAME}")
    //     }
    //   }
    // }
    // stage('Deploy Production') {
    //   // Production branch
    //   when { branch 'master' }
    //   steps{
    //     container('kubectl') {
    //     // Change deployed image in canary to the one we just built
    //       sh("sed -i.bak 's#docker.io/arifpradana22/gceme:1.0.0#${IMAGE_TAG}#' ./k8s/production/*.yaml")
    //       step([$class: 'KubernetesEngineBuilder', namespace:'production', projectId: env.PROJECT, clusterName: env.CLUSTER, zone: env.CLUSTER_ZONE, manifestPattern: 'k8s/services', credentialsId: env.JENKINS_CRED, verifyDeployments: false])
    //       step([$class: 'KubernetesEngineBuilder', namespace:'production', projectId: env.PROJECT, clusterName: env.CLUSTER, zone: env.CLUSTER_ZONE, manifestPattern: 'k8s/production', credentialsId: env.JENKINS_CRED, verifyDeployments: true])
    //       sh("echo http://`kubectl --namespace=production get service/${FE_SVC_NAME} -o jsonpath='{.status.loadBalancer.ingress[0].ip}'` > ${FE_SVC_NAME}")
    //     }
    //   }
    // }
    // stage('Deploy Dev') {
    //   // Developer Branches
    //   when {
    //     not { branch 'master' }
    //     not { branch 'canary' }
    //   }
    //   steps {
    //     container('kubectl') {
    //       // Create namespace if it doesn't exist
    //       sh("kubectl get ns ${env.BRANCH_NAME} || kubectl create ns ${env.BRANCH_NAME}")
    //       // Don't use public load balancing for development branches
    //       sh("sed -i.bak 's#LoadBalancer#ClusterIP#' ./k8s/services/frontend.yaml")
    //       sh("sed -i.bak 's#docker.io/arifpradana22/gceme:1.0.0#${IMAGE_TAG}#' ./k8s/dev/*.yaml")
    //       step([$class: 'KubernetesEngineBuilder', namespace: "${env.BRANCH_NAME}", projectId: env.PROJECT, clusterName: env.CLUSTER, zone: env.CLUSTER_ZONE, manifestPattern: 'k8s/services', credentialsId: env.JENKINS_CRED, verifyDeployments: false])
    //       step([$class: 'KubernetesEngineBuilder', namespace: "${env.BRANCH_NAME}", projectId: env.PROJECT, clusterName: env.CLUSTER, zone: env.CLUSTER_ZONE, manifestPattern: 'k8s/dev', credentialsId: env.JENKINS_CRED, verifyDeployments: true])
    //       echo 'To access your environment run `kubectl proxy`'
    //       echo "Then access your service via http://localhost:8001/api/v1/proxy/namespaces/${env.BRANCH_NAME}/services/${FE_SVC_NAME}:80/"
    //     }
    //   }
    // }
  }
  post{
    always{
      container('docker'){
        sh "docker logout"
      }  
    }
  }
}
