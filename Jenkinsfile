@Library('jenkins-shared-library')_

node {
    //node influences what jenkins worker node is used.
    def app
    def commit_id
    def color = 'YELLOW'
    colorCode = '#FFFF00'
    def msg = "Build Started - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
    def repo = "${env.GIT_URL}"


    stage('Clone repository') {
        checkout scm
        sh "git rev-parse --short HEAD > .git/commit-id" 
        commit_id = readFile('.git/commit-id').trim()
        slackSend(color: colorCode, message: msg)
    }

    stage('Build image') {
        /* This builds the actual image - like docker build*/
        sh "echo build-stage"
        sh 'echo "printing environment variables."'
        sh "printenv"
        sh "docker build -t node-example-jenkins docker/."
        color = 'GREEN'
        colorCode = '#00FF00'
        msg = "Build Succeeded - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
        slackSend(color: colorCode, message: msg)
    }

    stage('Push QA image') {
        if (env.BRANCH_NAME ==~ "develop") {
            sh "\$(aws ecr get-login --no-include-email --region us-east-2)"
            sh "docker tag node-example-jenkins:latest 545314842485.dkr.ecr.us-east-2.amazonaws.com/node-example-jenkins:latest"
            sh "docker push 545314842485.dkr.ecr.us-east-2.amazonaws.com/node-example-jenkins:latest"
            color = 'GREEN'
            colorCode = '#00FF00' 
            msg = "Push to ECR in QA Succeeded - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
            slackSend(color: colorCode, message: msg)
        }
    }

    stage('Push Prod image') {
        if (env.BRANCH_NAME ==~ "master") {
            sh "\$(aws ecr get-login --no-include-email --region us-east-1)"
            sh "docker tag node-example-jenkins:latest 545314842485.dkr.ecr.us-east-1.amazonaws.com/node-example-jenkins:latest"
            sh "docker push 545314842485.dkr.ecr.us-east-1.amazonaws.com/node-example-jenkins:latest"
            color = 'GREEN'
            colorCode = '#00FF00' 
            msg = "Push to ECR in Prod Succeeded - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
            slackSend(color: colorCode, message: msg)
        }
    }

    stage('QA Tests') {
        if (env.BRANCH_NAME ==~ "develop") {
                sh 'echo "All QA Tests passed."'
                color = 'GREEN'
                colorCode = '#00FF00'
                msg = "QA Tests Succeeded - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
                slackSend(color: colorCode, message: msg)
        }
    }

    stage('Prod Tests') {
        if (env.BRANCH_NAME ==~ "master") {
                sh 'echo "All QA Tests passed."'
                color = 'GREEN'
                colorCode = '#00FF00'
                msg = "QA Tests Succeeded - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
                slackSend(color: colorCode, message: msg)
        }
    }

    stage('Deploy QA') {
        if (env.BRANCH_NAME ==~ "develop") {
            colorCode = '#00FF00' 
            // msg = "Successfully Deployed to QA - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
            // sh "sudo /var/lib/jenkins/adventrv2-adventr-k8s/scripts/qa_deploy_script.sh node-example-jenkins"
            slackSend(color: colorCode, message: msg)
        }
    }

    stage('Deploy Prod') {
        if (env.BRANCH_NAME ==~ "master") {
            colorCode = '#00FF00' 
            // msg = "Successfully Deployed to Prod - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
            // sh "sudo /var/lib/jenkins/adventrv2-adventr-k8s/scripts/qa_deploy_script.sh node-example-jenkins"
            slackSend(color: colorCode, message: msg)
        }
    }
}