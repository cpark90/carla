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
            discordSend description: "알림테스트", 
            footer: "테스트 빌드가 성공했습니다.", 
            link: env.BUILD_URL, result: currentBuild.currentResult, 
            title: "테스트 젠킨스 job", 
            webhookURL: env.WEBHOOK_URL
        }
        failure {
            discordSend description: "알림테스트", 
            footer: "테스트 빌드가 실패했습니다.", 
            link: env.BUILD_URL, result: currentBuild.currentResult, 
            title: "테스트 젠킨스 job", 
            webhookURL: env.WEBHOOK_URL
        }
    }
}
