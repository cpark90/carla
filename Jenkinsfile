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
            agent { label "built-in" }
            steps
            {
                script
                {
                    JOB_ID = "${env.BUILD_TAG}"
                    echo "no job"
                }
            }
        }
        stage('Building CARLA')
        {
            parallel
            {
                stage('ubuntu')
                {
                    agent { label "sim125" }
                    environment
                    {
                        UE4_ROOT = '/home/jenkins/UnrealEngine_4.26'
                    }
                    stages
                    {
                        stage('ubuntu setup')
                        {
                            steps
                            {
                                sh 'git update-index --skip-worktree Unreal/CarlaUE4/CarlaUE4.uproject'
                                sh 'make setup ARGS="--python-version=3.7,2 --target-wheel-platform=manylinux_2_27_x86_64 --chrono"'
                            }
                        }
                        stage('ubuntu build')
                        {
                            steps
                            {
                                sh 'make LibCarla'
                                sh 'make PythonAPI ARGS="--python-version=3.7,2 --target-wheel-platform=manylinux_2_27_x86_64"'
                                sh 'make CarlaUE4Editor ARGS="--chrono"'
                                sh 'make plugins'
                                sh 'make examples'
                            }
                            post
                            {
                                always
                                {
                                    archiveArtifacts 'PythonAPI/carla/dist/*.egg'
                                    archiveArtifacts 'PythonAPI/carla/dist/*.whl'
                                    stash includes: 'PythonAPI/carla/dist/*.egg', name: 'ubuntu_eggs'
                                    stash includes: 'PythonAPI/carla/dist/*.whl', name: 'ubuntu_wheels'
                                }
                            }
                        }
                        stage('ubuntu unit tests')
                        {
                            steps
                            {
                                sh 'make check ARGS="--all --xml --python-version=3.7,2 --target-wheel-platform=manylinux_2_27_x86_64"'
                            }
                            post
                            {
                                always
                                {
                                    junit 'Build/test-results/*.xml'
                                    archiveArtifacts 'profiler.csv'
                                }
                            }
                        }
                        stage('ubuntu retrieve content')
                        {
                            steps
                            {
                                sh './Update.sh'
                            }
                        }
                        stage('ubuntu package')
                        {
                            steps
                            {
                                sh 'make package ARGS="--python-version=3.7,2 --target-wheel-platform=manylinux_2_27_x86_64 --chrono"'
                                sh 'make package ARGS="--packages=AdditionalMaps,Town06_Opt,Town07_Opt,Town11 --target-archive=AdditionalMaps --clean-intermediate --python-version=3.7,2 --target-wheel-platform=manylinux_2_27_x86_64"'
                                sh 'make examples ARGS="localhost 3654"'
                            }
                            post
                            {
                                always
                                {
                                    archiveArtifacts 'Dist/*.tar.gz'
                                    stash includes: 'Dist/CARLA*.tar.gz', name: 'ubuntu_package'
                                    // stash includes: 'Dist/AdditionalMaps*.tar.gz', name: 'ubuntu_package2'
                                    stash includes: 'Examples/', name: 'ubuntu_examples'
                                }
                                success
                                {
                                    node('master')
                                    {
                                        script
                                        {
                                            echo "create node"
                                        }
                                    }
                                }
                            }
                        }
                        stage('ubuntu smoke tests')
                        {
                            agent { label "sim_gpu" }
                            steps
                            {
                                unstash name: 'ubuntu_eggs'
                                unstash name: 'ubuntu_wheels'
                                unstash name: 'ubuntu_package'
                                // unstash name: 'ubuntu_package2'
                                unstash name: 'ubuntu_examples'
                                sh 'tar -xvzf Dist/CARLA*.tar.gz -C Dist/'
                                // sh 'tar -xvzf Dist/AdditionalMaps*.tar.gz -C Dist/'
                                sh 'DISPLAY= ./Dist/CarlaUE4.sh -nullrhi -RenderOffScreen --carla-rpc-port=3654 --carla-streaming-port=0 -nosound > CarlaUE4.log &'
                                sh 'make smoke_tests ARGS="--xml --python-version=3.7,2 --target-wheel-platform=manylinux_2_27_x86_64"'
                                sh 'make run-examples ARGS="localhost 3654"'
                            }
                            post
                            {
                                always
                                {
                                    archiveArtifacts 'CarlaUE4.log'
                                    junit 'Build/test-results/smoke-tests-*.xml'
                                    deleteDir()
                                    node('master')
                                    {
                                        echo "delete node"
                                    }
                                }
                            }
                        }
                        stage('ubuntu deploy dev')
                        {
                            when { branch "dev"; }
                            steps
                            {
                                sh 'git checkout .'
                                sh 'make deploy ARGS="--replace-latest"'
                            }
                        }
                        stage('ubuntu deploy master')
                        {
                            when { anyOf { branch "master"; buildingTag() } }
                            steps
                            {
                                sh 'git checkout .'
                                sh 'make deploy ARGS="--replace-latest --docker-push"'
                            }
                        }
                        stage('ubuntu Doxygen')
                        {
                            when { anyOf { branch "master"; branch "dev"; buildingTag() } }
                            steps
                            {
                                sh 'rm -rf ~/carla-simulator.github.io/Doxygen'
                                sh '''
                                    cd ~/carla-simulator.github.io
                                    git remote set-url origin git@docs:carla-simulator/carla-simulator.github.io.git
                                    git fetch
                                    git checkout -B master origin/master
                                '''
                                sh 'make docs'
                                sh 'cp -rf ./Doxygen ~/carla-simulator.github.io/'
                                sh '''
                                    cd ~/carla-simulator.github.io
                                    git add Doxygen
                                    git commit -m "Updated c++ docs" || true
                                    git push
                                '''
                            }
                            post
                            {
                                always
                                {
                                    deleteDir()
                                }
                            }
                        }
                    }
                    post
                    {
                        always
                        {
                            deleteDir()

                            node('master')
                            {
                                script
                                {
                                    echo "delete node"
                                }
                            }
                        }
                    }
                }
                stage('windows')
                {
                    agent { label "sim122" }
                    stages
                    {
                        stage('windows setup')
                        {
                            steps
                            {
                                bat """
                                    call ../setEnv64.bat
                                    git update-index --skip-worktree Unreal/CarlaUE4/CarlaUE4.uproject
                                """
                                bat """
                                    call ../setEnv64.bat
                                    make setup ARGS="--chrono"
                                """
                            }
                        }
                        stage('windows build')
                        {
                            steps
                            {
                                bat """
                                    call ../setEnv64.bat
                                    make LibCarla
                                """
                                bat """
                                    call ../setEnv64.bat
                                    make PythonAPI
                                """
                                bat """
                                    call ../setEnv64.bat
                                    make CarlaUE4Editor ARGS="--chrono"
                                """
                                bat """
                                    call ../setEnv64.bat
                                    make plugins
                                """
                            }
                            post
                            {
                                always
                                {
                                    archiveArtifacts 'PythonAPI/carla/dist/*.egg'
                                    archiveArtifacts 'PythonAPI/carla/dist/*.whl'
                                }
                            }
                        }
                        stage('windows retrieve content')
                        {
                            steps
                            {
                                bat """
                                    call ../setEnv64.bat
                                    call Update.bat
                                """
                            }
                        }
                        stage('windows package')
                        {
                            steps
                            {
                                bat """
                                    call ../setEnv64.bat
                                    make package ARGS="--chrono"
                                """
                                bat """
                                    call ../setEnv64.bat
                                    make package ARGS="--packages=AdditionalMaps,Town06_Opt,Town07_Opt,Town11 --target-archive=AdditionalMaps --clean-intermediate"
                                """
                            }
                            post {
                                always {
                                    archiveArtifacts 'Build/UE4Carla/*.zip'
                                }
                            }
                        }
                        stage('windows deploy')
                        {
                            when { anyOf { branch "master"; branch "dev"; buildingTag() } }
                            steps {
                                bat """
                                    call ../setEnv64.bat
                                    git checkout .
                                    make deploy ARGS="--replace-latest"
                                """
                            }
                        }
                    }
                    post
                    {
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
                        always
                        {
                            deleteDir()

                            node('built-in')
                            {
                                script
                                {

                                    echo env.BUILD_URL
                                    echo currentBuild.currentResult
                                    echo env.WEBHOOK_URL
                                    echo "no job"
                                }
                            }
                        }
                    }
                }
            }
        }
    }
}