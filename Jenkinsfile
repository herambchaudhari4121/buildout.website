@Library('jenkins-pipeline-scripts') _
pipeline {
    agent none
    options {
        // Keep the 50 most recent builds
        buildDiscarder(logRotator(numToKeepStr:'50'))
    }
    stages {
        stage('Build') {
            agent any
            steps {
                sh 'make eggs'
                sh 'make docker-image'
            }
        }
        stage('Push image to registry') {
            agent any
            steps {
                pushImageToRegistry (
                    env.BUILD_ID,
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
                sh "mco shell run 'docker pull docker-staging.imio.be/iasmartweb/mutual:$BUILD_ID' -I /^site-staging/ -I /^staging.imio.be/"
                sh "mco shell run '/srv/docker_scripts/website-update-all-images.sh' -t 1200 --tail -I /^site-staging/ -I /^staging.imio.be/ "
            }
        }
    }
}
