def frontendImage="bsz87/frontend"
def backendImage="bsz87/backend"
def dockerRegistry=""
def registryCredentials="dockerhub"
def backendDockerTag=""
def frontendDockerTag=""

pipeline {
    agent {
        label 'agent'
    }
    tools {
        terraform 'Terraform'
    }
    environment {
        PIP_BREAK_SYSTEM_PACKAGES = 1
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
        stage('Adjust version') {
            steps {
                script{
                    backendDockerTag = params.backendDockerTag.isEmpty() ? "latest" : params.backendDockerTag
                    frontendDockerTag = params.frontendDockerTag.isEmpty() ? "latest" : params.frontendDockerTag
                    
                    currentBuild.description = "Backend: ${backendDockerTag}, Frontend: ${frontendDockerTag}"
                }
            }
        }
        stage('Clean environment') {
            steps {
                sh "docker rm -f frontend backend"
            }
        }
        stage('Deploy application') {
            steps {
                script {
                    withEnv(["FRONTEND_IMAGE=$frontendImage:$frontendDockerTag", 
                             "BACKEND_IMAGE=$backendImage:$backendDockerTag"]) {
                       docker.withRegistry("$dockerRegistry", "$registryCredentials") {
                            sh "docker-compose up -d"
                        }
                    }
                }
            }
        }
        stage('Selenium tests') {
            steps {
                sh "pip3 install -r test/selenium/requirements.txt"
                sh "python3 -m pytest test/selenium/seleniumTest.py"
            }
        }
        stage('Run terraform') {
            steps {
                dir('Terraform') {                
                    git branch: 'main', url: 'https://github.com/bsz87/terraform'
                    withAWS(credentials:'AWS', region: 'us-east-1') {
                            sh 'terraform init -backend-config=bartlomiej-szelagowski-panda-devops-core-15'
                            sh 'terraform apply -auto-approve -var bucket_name=bartlomiej-szelagowski-panda-devops-core-15'
                            
                    } 
                }
            }
        }
        stage('Run Ansible') {
            steps {
                script {
                    sh "pip3 install -r requirements.txt"
                    sh "ansible-galaxy install -r requirements.yml"
                    withEnv(["FRONTEND_IMAGE=$frontendImage:$frontendDockerTag", 
                            "BACKEND_IMAGE=$backendImage:$backendDockerTag"]) {
                            ansiblePlaybook inventory: 'inventory', playbook: 'playbook.yml'
                    }
                }
            }
        }
    }
    post {
        always {
            sh "docker-compose down"
            cleanWs()
        }
    }
}