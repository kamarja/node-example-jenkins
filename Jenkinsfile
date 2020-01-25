#!groovy
@Library('jenkins-shared-library')_

pipeline {
  agent any

  stages {

    stage('Preparing') {
        environment {
            //commit_id = readFile('.git/commit-id').trim()
            msg = "Build Started - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
            qa_ecr_msg = "Push to ECR in QA Succeeded - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
            prod_ecr_msg = "Push to ECR in Prod Succeeded - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
            repo = "${env.GIT_URL}"
            server_name = "${env.HUDSON_URL}"
            branch = 'develop'
            REGION = sh(returnStdout: true, script: 'curl -s http://169.254.169.254/latest/dynamic/instance-identity/document | jq -r .region').trim()
        }

        steps {
            sh 'echo "Prepared variables."'
            //sh "git rev-parse --short HEAD > .git/commit-id" 
            //slackSend(color: colorCode, message: msg)
              script {
                   def fields = env.getEnvironment()
                   fields.each {
                        key, value -> println("${key} = ${value}");
                    }
 
                    println(env.REGION)
               }
        }
    }

    stage('Clone repository') {
        steps {
            checkout scm
            //sh "git rev-parse --short HEAD > .git/commit-id" 
            //slackSend(color: colorCode, message: msg)
        }
    }

    stage('Build image') {
        environment {
            msg = "Build Succeeded - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
        }  

        steps {
            /* This builds the actual image - like docker build*/
            sh "echo build-stage"
            sh 'echo "Printing environment variables."'
            sh "printenv"
            sh "docker build -t node-example-jenkins docker/."
            //slackSend(color: colorCode, message: msg)       
        }

    }

    stage('Push QA image') {
        environment {
            server_name = "${env.HUDSON_URL}"
            branch = 'develop'
            REGION = sh (returnStdout: true, script: 'curl -s http://169.254.169.254/latest/dynamic/instance-identity/document | jq -r .region').trim()
        }

        when { equals expected: 'develop', actual: 'develop' }

        // when {
        //         beforeAgent true
        //         //env.HUDSON_URL 'http://jenkins.theadventr.com:8080/'
        //         branch 'develop'
        // }

        steps {
            sh 'echo "Pushing QA image."'
            sh 'echo $REGION'
            sh "\$(aws ecr get-login --no-include-email --region us-east-2)"
            sh "docker tag node-example-jenkins:latest 545314842485.dkr.ecr.us-east-2.amazonaws.com/node-example-jenkins:latest"
            sh "docker push 545314842485.dkr.ecr.us-east-2.amazonaws.com/node-example-jenkins:latest"
            //slackSend(color: colorCode, message: msg)     
        }
    }

    stage('Push Prod image') {
        when {
                beforeAgent true
                //env.HUDSON_URL 'http://jenkins.adventr.me:8080/'
                branch 'master'
        }

        steps {
            sh 'echo "Pushing QA image."'
            // sh "\$(aws ecr get-login --no-include-email --region us-east-1)"
            // sh "docker tag node-example-jenkins:latest 545314842485.dkr.ecr.us-east-1.amazonaws.com/node-example-jenkins:latest"
            // sh "docker push 545314842485.dkr.ecr.us-east-1.amazonaws.com/node-example-jenkins:latest"
            // environment {
            //     msg = "Push to ECR in Prod Succeeded - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
            // }
            //slackSend(color: colorCode, message: msg)       
        }
    }
  }
}