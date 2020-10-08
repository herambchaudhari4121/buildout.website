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
    }
}
