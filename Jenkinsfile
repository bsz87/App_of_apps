def frontendImage="bsz87/frontend"
def backendImage="bsz87/backend"
def dockerRegistry=""
def registryCredentials="dockerhub" 

pipeline {
    agent {
        label 'agent'
    }

    stages {
        stage('Get Code') {
            steps {
                checkout scm
            }
        }
    }
}