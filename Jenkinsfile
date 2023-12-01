def frontendImage="bsz87/frontend"
def backendImage="bsz87/backend"
def dockerRegistry=""
def registryCredentials="dockerhub"

pipeline {
    agent {
        label 'agent'
    }
    parameters {
        string(name: 'backendDockerTag', defaultValue: '', description: 'Backend docker image tag')
        string(name: 'frontendDockerTag', defaultValue: '', description: 'Frontend docker image tag')
    }
    stages {
        stage('Get Code') {
            steps {
                checkout scm
            }
        }
    }
}