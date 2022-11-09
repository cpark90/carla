#!/usr/bin/env groovy
@Library('carla_cpark') _ 

pipeline
{
    agent none

    options
    {
        buildDiscarder(logRotator(numToKeepStr: '3', artifactNumToKeepStr: '3'))
    }

    stages
    {
        stage('Creating nodes')
        {
            sendNotifications 'STARTED'
        }
    }
}
