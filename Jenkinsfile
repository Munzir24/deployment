pipeline {
    agent {
        label "slave1"
    }
    parameters {
        string (name: 'VERSION', defaultValue: '0.0.0', description: 'Version to be deployed')
        choice (name: 'ENVIRONMENT', choices: ['dev','prod','SIT'], description: 'Environment on which to be deployed')
    }
    environment {
        ANSIBLE_INVENTORY = 'hosts'
        CONFIGURE_PLAYBOOK = 'mytomcat.yaml'
        DEPLOY_PLAYBOOK = 'deploy-cicd.yaml'
        GIT_REPO = 'https://github.com/Munzir24/deployment.git'
        SSH_KEY_PATH = '/home/ec2-user/.ssh/mykey.pem'
        ANSIBLE_EXTRA_ARGS = "--private-key ${SSH_KEY_PATH}"
        NEXUS_CREDENTIALS = credentials('nexus-token')
    }
    stages {
        stage('Install Python3') {
            steps {
                script {
                    sh '''
                    if ! command -v python3 &> /dev/null; then
                        sudo yum install epel-release -y
                        sudo yum install python3 -y
                    fi '''
                    }
                }
            }
        stage('Clone Code from Git Repository') {
            steps {
                script {
                    git url: "${env.GIT_REPO}"
                }
            }
        }
        stage('Configure JAVA and TOMCAT on desired environment') {
            steps {
                script {
                    ansiblePlaybook(
                        playbook: "${env.CONFIGURE_PLAYBOOK}",
                        inventory: "${env.ANSIBLE_INVENTORY}",
                        extras: "-e env=${params.ENVIRONMENT} ${env.ANSIBLE_EXTRA_ARGS}"
                    )
                }
            }
        }
        stage('Deploy Artifact to the desired environment') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'nexus-token', passwordVariable: 'NEXUS_PASSWORD', usernameVariable: 'NEXUS_USER')])
                script {
                    ansiblePlaybook(
                        playbook: "${env.DEPLOY_PLAYBOOK}",
                        inventory: "${env.ANSIBLE_INVENTORY}",
                        extras: "-e \"env=${params.ENVIRONMENT} nexus_user=${NEXUS_USER} nexus_password=${NEXUS_PASSWORD} version=${params.VERSION} \" ${env.ANSIBLE_EXTRA_ARGS}"
                    )
                }
            }
        }
    }
}
