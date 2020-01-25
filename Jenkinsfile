#!groovy
@Library('jenkins-shared-library')_

pipeline {
  agent none
  environment {
    //commit_id = readFile('.git/commit-id').trim()
    msg = "Build Started - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
    repo = "${env.GIT_URL}"
    server_name = "${env.HUDSON_URL}"

  }
  stages {
    stage('Clone repository') {
        steps {
            checkout scm
            //sh "git rev-parse --short HEAD > .git/commit-id" 
            
            //slackSend(color: colorCode, message: msg)
        }
    }

    stage('Build image') {
        steps {
            /* This builds the actual image - like docker build*/
            sh "echo build-stage"
            sh 'echo "Printing environment variables."'
            sh "printenv"
            sh 'echo "Printing Jenkins Internal Variables"'
            script {
                fields.each {
                    key, value -> println("${key} = ${value}");
                }
            }
            sh "docker build -t node-example-jenkins docker/."
            color = 'GREEN'
            colorCode = '#00FF00'
            msg = "Build Succeeded - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
            //slackSend(color: colorCode, message: msg)       
        }

    }

    stage('Push QA image') {
        steps {
            when {
                beforeAgent true
                environment name: 'HUDSON_URL', value: 'http://jenkins.theadventr.com:8080/'
            }

            sh 'echo "Pushing QA image."'
            sh "\$(aws ecr get-login --no-include-email --region us-east-2)"
            sh "docker tag node-example-jenkins:latest 545314842485.dkr.ecr.us-east-2.amazonaws.com/node-example-jenkins:latest"
            sh "docker push 545314842485.dkr.ecr.us-east-2.amazonaws.com/node-example-jenkins:latest"
            color = 'GREEN'
            colorCode = '#00FF00' 
            msg = "Push to ECR in QA Succeeded - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
            //slackSend(color: colorCode, message: msg)
                
            }
    }

    stage('Push Prod image') {
        steps {
            when {
                beforeAgent true
                environment name: 'HUDSON_URL', value: 'http://jenkins.adventr.me:8080/'
            }

            sh 'echo "Pushing QA image."'
            sh "\$(aws ecr get-login --no-include-email --region us-east-1)"
            sh "docker tag node-example-jenkins:latest 545314842485.dkr.ecr.us-east-1.amazonaws.com/node-example-jenkins:latest"
            sh "docker push 545314842485.dkr.ecr.us-east-1.amazonaws.com/node-example-jenkins:latest"
            color = 'GREEN'
            colorCode = '#00FF00' 
            msg = "Push to ECR in Prod Succeeded - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
            //slackSend(color: colorCode, message: msg)       
        }
    }
  }
}