@Library('jenkins-pipeline-scripts') _
pipeline {
    agent none
    triggers {
        pollSCM('*/3 * * * *')
    }
    options {
        // Keep the 50 most recent builds
        buildDiscarder(logRotator(numToKeepStr:'50'))
    }
    stages {
        stage('Build') {
            agent any
            steps {
                sh 'make docker-image'
            }
        }
        stage('Build dependencies') {
            agent any
            steps {
                parallel (
                    "ideabox": {
                        echo "starting ideabox build"
                        build job: '/IMIO-github-Jenkinsfile/buildout.ideabox/master', wait: false
                    }
                )
            }
        }
        stage('Push image to registry') {
            agent any
            steps {
                pushImageToRegistry (
                    $BUILD_ID,
                    "iasmartweb/mutual"
                )
            }
        }
        stage('Deploy to staging') {
            agent any
            when {
                expression {
                    currentBuild.result == null || currentBuild.result == 'SUCCESS'
                }
            }
            steps {
                sh "mco shell run 'docker pull docker-staging.imio.be/iasmartweb/mutual:$BUILD_ID' -I /^staging.imio.be/"
                sh "mco shell run '/srv/docker_scripts/website-update-all-images.sh' -t 1200 --tail -I /^staging.imio.be/"
            }
        }
        stage('Deploy to prod ?') {
            agent none
            steps {
                timeout(time: 24, unit: 'HOURS') {
                    input (
                        message: 'Should we deploy to prod ?'
                    )
                }
            }
            post {
                aborted {
                    echo 'In post aborted'
                }
                success {
                    echo 'In post success'
                }
            }
        }
        stage('Deploying to prod') {
            agent any
            steps {
                deployToProd (
                    $BUILD_ID,
                    "iasmartweb/mutual",
                    "C /role::docker::sites$/",
                    '/srv/docker_scripts/website-update-all-images.sh',
                )
            }
        }
    }
}
