#! /usr/bin/env groovy

pipeline {

  agent {
    label 'maven'
  }

  stages {
    stage('Build') {
      steps {
        echo 'Building..'
          sh 'mvn clean package'
      }
    }
    stage('Create Container Image') {
      steps {
        echo 'Create Container Image..'
        
        script {
openshift.withCluster() { 
  openshift.withProject("microservices-oc") {
  
    def buildConfigExists = openshift.selector("bc", "springboot-pipeline").exists() 
    
    if(!buildConfigExists){ 
      openshift.newBuild("--name=springboot-pipeline", "--docker-image=docker.io/centos:7", "--binary") 
    } 
    
    openshift.selector("bc", "springboot-pipeline").startBuild("--from-file=target/helloworld-0.0.1-SNAPSHOT.jar", "--follow") } }

        }
      }
    }
    stage('Deploy') {
      steps {
        echo 'Deploying....'
        script {

         openshift.withCluster() { 
  openshift.withProject("microservices-oc") { 
    def deployment = openshift.selector("dc", "springboot-pipeline") 
    
    if(!deployment.exists()){ 
      openshift.newApp('springboot-pipeline', "--as-deployment-config").narrow('svc').expose() 
    } 
    
    timeout(5) { 
      openshift.selector("dc", "springboot-pipeline").related('pods').untilEach(1) { 
        return (it.object().status.phase == "Running") 
      } 
    } 
  } 
}

        }
      }
    }
  }
}