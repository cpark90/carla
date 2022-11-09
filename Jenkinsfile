#!/usr/bin/env groovy
@Library('cpark_carla') _ 

pipeline
{
    agent none
    environment {
        WEBHOOK_URL = credentials("DISCORD_WEBHOOK")
    }
    options
    {
        buildDiscarder(logRotator(numToKeepStr: '3', artifactNumToKeepStr: '3'))
    }
    stages
    {
        stage('Creating nodes')
        {
            steps{
                echo env.WEBHOOK_URL
            }
        }
    }
    post {
        success {
            discordSend description: "Notification test", 
            footer: "Test build success", 
            link: env.BUILD_URL, result: currentBuild.currentResult, 
            title: "Test jenkins job", 
            webhookURL: env.WEBHOOK_URL
        }
        failure {
            discordSend description: "Notification test", 
            footer: "Test build fail", 
            link: env.BUILD_URL, result: currentBuild.currentResult, 
            title: "Test jenkins job", 
            webhookURL: env.WEBHOOK_URL
        }
    }
}
