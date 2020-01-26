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
            REGION = sh(returnStdout: true, script: 'curl -s http://169.254.169.254/latest/dynamic/instance-identity/document | jq -r .region').trim()
            branch = getContext(REGION)
        }

        steps {
            sh 'echo "Prepared variables."'
            initialize()
            //sh "git rev-parse --short HEAD > .git/commit-id" 
            //slackSend(color: colorCode, message: msg)
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
            server_name = "${env.HUDSON_URL}"
            REGION = sh(returnStdout: true, script: 'curl -s http://169.254.169.254/latest/dynamic/instance-identity/document | jq -r .region').trim()
            branch = getContext(REGION)
        }  

        steps {
            /* This builds the actual image - like docker build*/
            sh "echo build-stage"
            sh 'echo "Printing environment variables."'
            sh 'echo $REGION'
            sh 'echo $server_name'
            sh "docker build -t node-example-jenkins docker/."
            //slackSend(color: colorCode, message: msg)       
        }

    }

    stage('Push QA image') {
        environment {
            server_name = "${env.HUDSON_URL}"
            REGION = sh(returnStdout: true, script: 'curl -s http://169.254.169.254/latest/dynamic/instance-identity/document | jq -r .region').trim()
            branch = getContext(REGION)
        }

        when {
            beforeInput true
            equals expected: env.HUDSON_URL, actual: 'http://jenkins-qa.theadventr.com:8080/' 
        }

        steps {
            sh 'echo "Pushing QA image."'
            sh "\$(aws ecr get-login --no-include-email --region us-east-2)"
            sh "docker tag node-example-jenkins:latest 545314842485.dkr.ecr.us-east-2.amazonaws.com/node-example-jenkins:latest"
            sh "docker push 545314842485.dkr.ecr.us-east-2.amazonaws.com/node-example-jenkins:latest"
            //slackSend(color: colorCode, message: msg)     
        }
    }

    stage('Push Prod image') {
        environment {
            server_name = "${env.HUDSON_URL}"
            REGION = sh(returnStdout: true, script: 'curl -s http://169.254.169.254/latest/dynamic/instance-identity/document | jq -r .region').trim()
            branch = getContext(REGION)
        }

        when {
            beforeInput true 
            equals expected: env.HUDSON_URL, actual: 'http://jenkins.adventr.me:8080/' 
        }

        steps {
            sh 'echo "Pushing Prod image."'
            sh "\$(aws ecr get-login --no-include-email --region us-east-1)"
            sh "docker tag node-example-jenkins:latest 545314842485.dkr.ecr.us-east-1.amazonaws.com/node-example-jenkins:latest"
            sh "docker push 545314842485.dkr.ecr.us-east-1.amazonaws.com/node-example-jenkins:latest"
            //slackSend(color: colorCode, message: msg)       
        }
    }

    stage('Deploy QA') {
        environment {
            server_name = "${env.HUDSON_URL}"
            REGION = sh(returnStdout: true, script: 'curl -s http://169.254.169.254/latest/dynamic/instance-identity/document | jq -r .region').trim()
            branch = getContext(REGION)
        }

        when {
            beforeInput true
            equals expected: env.HUDSON_URL, actual: 'http://jenkins-qa.theadventr.com:8080/' 
        }

        steps {
            sh 'echo "Demploying QA image."'
            // msg = "Successfully Deployed to QA - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
            // sh "sudo /var/lib/jenkins/adventrv2-adventr-k8s/scripts/qa_deploy_script.sh node-example-jenkins"
            //slackSend(color: colorCode, message: msg)      
        }
    }

    stage('Deploy Prod') {
        environment {
            server_name = "${env.HUDSON_URL}"
            REGION = sh(returnStdout: true, script: 'curl -s http://169.254.169.254/latest/dynamic/instance-identity/document | jq -r .region').trim()
            branch = getContext(REGION)
        }

        when {
            beforeInput true
            equals expected: env.HUDSON_URL, actual: 'http://jenkins.adventr.me:8080/' 
        }

        steps {
            sh 'echo "Demploying Prod image."'
            // msg = "Successfully Deployed to Prod - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
            // sh "sudo /var/lib/jenkins/adventrv2-adventr-k8s/scripts/qa_deploy_script.sh node-example-jenkins"
            //slackSend(color: colorCode, message: msg)      
        }
    }
  }
}

// ================================================================================================
// Scripts
// ================================================================================================

def getContext(region) { 
    (env.BRANCH_NAME == 'develop') ? region : 'us-east-2'
    (env.BRANCH_NAME == 'master') ? region : 'us-east-1'
    return env.BRANCH_NAME
}

def showEnvironmentVariables() {
    sh 'env | sort > env.txt'
    sh 'cat env.txt'
}

def initialize() {
    sh 'branch=$(echo "$GIT_BRANCH")'
    sh 'curl -s http://169.254.169.254/latest/dynamic/instance-identity/document | jq -r .region > region.txt'
    AWS_REGION = readFile 'region.txt'
    echo "This is the region: ${AWS_REGION}"
    getContext("${AWS_REGION}")
    //Enable the function below for debbugging.
    showEnvironmentVariables()
}


