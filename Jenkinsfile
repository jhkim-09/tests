pipeline {
  agent {
    kubernetes {
      yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: maven
    image: maven:3.8.6-openjdk-8
    command:
    - sleep
    args:
    - infinity
  - name: git
    image: alpine/git
    command:
    - sleep
    args:
    - infinity
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    command:
    - sleep
    args:
    - infinity
    volumeMounts:
    - name: registry-credentials
      mountPath: /kaniko/.docker
  volumes:
  - name: registry-credentials
    secret:
      secretName: regcred
      items:
      - key: .dockerconfigjson
        path: config.json
'''
    }
  }
  stages {
    stage('Checkout') {
      steps {
        container('maven') {
          git branch: 'main', url: 'https://github.com/jhkim-09/tests.git'
        }
      }
    }
    stage('Build') {
      steps {
        container('maven') {
          sh 'mvn clean package -DskipTests=true'
        }
      }
    }
    stage('Test') {
      steps {
        container('maven') {
          sh 'mvn test'
        }
      }
    }
    stage('Build & Tag & Push Docker Image') {
      steps {
        container('kaniko') {
          sh 'executor --context=dir:///home/jenkins/agent/workspace/kube_pipeline/ --destination=kimjuhyo/cicd-k8s:$BUILD_NUMBER --destination=kimjuhyo/cicd-k8s:latest'
        }
      }
    }
    stage('Update K8s Manifests & Push') {
      environment {
        githubUser = 'jhkim-09' 
        githubEmail = 'kimjuhyo@naver.com' //c1t1d0s7@example.com
        githubId = 'jhkim-09' //c1t1d0s7 
        githubRepo = 'tests' //jenkins-kube-deploy
        githubURL = "https://github.com/${githubId}/${githubRepo}.git"
        dockerhubId = 'kimjuhyo' //c1t1d0s7
        dockerhubRepo = 'cicd-k8s' //hello-world
      }
      steps {
        container('git') {
          git branch: 'main', credentialsId: 'github-credential', url: "${githubURL}"
          sh "git config --global --add safe.directory ${workspace}"
          sh "git config --global user.name ${githubUser}"
          sh "git config --global user.email ${githubEmail}"
          sh 'sed -i "s/image:.*/image: ${dockerhubId}\\/${dockerhubRepo}:${BUILD_NUMBER}/g" deployment.yaml'
          sh 'git add deployment.yaml'
          sh 'git commit -m "Jenkins Build Number - ${BUILD_NUMBER}"'
          withCredentials([gitUsernamePassword(credentialsId: 'github-credential', gitToolName: 'Default')]) {
            sh 'git push --set-upstream origin main'
          }
        }
      }
    }
  }
}
