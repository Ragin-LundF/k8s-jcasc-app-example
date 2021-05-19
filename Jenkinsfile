pipeline {
  options {
      buildDiscarder(logRotator(numToKeepStr: '5', artifactNumToKeepStr: '5'))
  }
  agent {
      label 'gradle_java_node'
  }

  stages {
    stage('Build Gradle') { steps { container(name: 'gradle') { script {
        sh "gradle build"
    } } } }

    stage('Build NPM') { steps { container(name: 'node') { script {
        dir('node-ui') {
            sh "npm ci install"
        }
    } } } }

    stage('Build Docker') { steps { container(name: 'docker') { script {
        withCredentials([usernamePassword(credentialsId: 'github-gcr-docker-login', passwordVariable: 'DOCKER_PWD', usernameVariable: 'DOCKER_USR')]) {
            sh "echo ${DOCKER_PWD} | docker login ghcr.io -u ${DOCKER_PWD} --password-stdin"
        }

        // build and push the image
        sh "docker build -t reddot:latest -f devops/Dockerfile ."
        sh "docker image tag reddot:latest ghcr.io/ragin-lundf/k8s-jcasc-app-example/reddot/reddot:latest"
        sh "docker image push ghcr.io/ragin-lundf/k8s-jcasc-app-example/reddot/reddot:latest"
    } } } }

    stage('Deploy: Dev') { steps { container(name: 'helm') { script {
        echo "deploying..."
        sh "wget --auth-no-challenge --http-user=admin --http-password=admin http://jenkins-controller:8080/jenkins/view/Demo%20deploy/job/Deploy%20DEMO%20dev/build?token=6f33625a-667e-4043-97f5-a8341eb3fa4b"
    } } } }

    stage('Test E2E: Dev') { steps { container(name: 'helm') { script {
        // imaging, that we are testing something here
        sleep(60)
    } } } }

    stage('Undeploy: Dev') { steps { container(name: 'helm') { script {
        echo "Undeploying..."
        sh "wget --auth-no-challenge --http-user=admin --http-password=admin http://jenkins-controller:8080/jenkins/view/Demo%20deploy/job/Undeploy%20DEMO%20dev/build?token=e858d2ac-e4b7-45ff-827c-8ce58e9f7dea"
    } } } }

  }
}