#!/usr/bin/env groovy
pipeline {

    /*
     * Run everything on an existing agent configured with a label 'docker'.
     * This agent will need docker, git, python and a jdk installed at a minimum.
     */
    agent {
        node {
            label 'docker'
        }
    }

    // using the Timestamper plugin we can add timestamps to the console log
    options {
        timestamps()
    }

    environment {
        //Use Pipeline Utility Steps plugin to read information from pom.xml into env variables
        DOCKERHUBID = swethabk92
        IMAGE = seniordevopsengineer-flask
        VERSION = "1.0.0"
    }

    stages {
        stage('Checkout') {
        script {
           // The below will clone your repo and will be checked out to master branch by default.
           git credentialsId: 'jenkins-user-github'           
           // List all branches in your repo. 
           sh "git branch -a"
           // Checkout to a specific branch in your repo.
           sh "git checkout seniordevopsengineer_task"
          }
            }
          }

             stage('Sonar Scan') {
                    environment {
                        //use 'sonar' credentials scoped only to this stage
                        SONAR = credentials('sonar')
                    }
                    steps {
                        sh 'mvn sonar:sonar -Dsonar.login=$SONAR_PSW'
                    }
                }
            

        stage('Build and Publish Image') {
            steps {
                sh """
              docker build -t ${IMAGE}:${VERSION} .
              docker tag ${IMAGE}:${VERSION} ${DOCKERHUBID}/${IMAGE}:${VERSION}
              docker push ${DOCKERHUBID}/${IMAGE}:${VERSION}
        """
            }
        }
            stage('Deploy AWS Cluster') {
            steps {
                 /*
                 *Creating all required securitygroups, Cluster using Makefile
                 */
                sh """
                make iam-securitygroup
                sleep 60
                make createCluster
                sleep 60
        """
            }
        }
                stage('Deploy Flask') {
            steps {
                 /*
                 *Creating all required configmaps and ClusterNodegroup using Makefile and deploy the flask pod
                 */
                sh """
                make applyConfigMap
                make CreateNodeGroup
                sleep 60
                make deploy
        """
            }
        }
}
