#! /usr/bin/env groovy

pipeline {

  agent {
    label 'maven'
  }

  stages {
    stage('Build') {
      steps {
        echo 'Building...'
          sh "mvn clean package"
      }
    }
    
        stage('Create Image Builder') {

            when {
              expression {
                openshift.withCluster() {
                  openshift.withProject("microservices-oc") {
                    return !openshift.selector("bc", "springpipe-hello").exists()
                  }
                }
              }
            }
            steps {
              script {
                openshift.withCluster() {
                  openshift.withProject("microservices-oc") {
                    openshift.newBuild("--name=javafabric8", "--docker-image=docker.io/fabric8/java-centos-openjdk11-jdk", "--binary=true")
                  }
                }
              }
            }
          }



          stage('Build Image') {
            steps {
              sh "rm -rf ocp && mkdir -p ocp/deployments"
              sh "pwd && ls -la target "
              sh "cp target/helloworld-*.jar ocp/deployments"

              script {
                openshift.withCluster() {
                  openshift.withProject("microservices-oc") {
                    openshift.selector("bc", "springpipe-hello").startBuild("--from-dir=./ocp","--follow", "--wait=true")
                  }
                }
              }
            }
          }

  }
}