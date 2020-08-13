pipeline {
    agent {
        docker { 
            image 'ubuntu:18.04' 
            args '-u root:sudo'
        }
    }
    stages {
        stage('Cleanup Workspace') {
            steps {
                cleanWs()
                sh """
                echo "Cleaned Up Workspace For Project"
                """
            }
        }
        stage('Install tools') {
            steps {
                sh '''
                set -x
                apt-get update -y
                apt-get install -y curl wget jq git
                '''
            }
        }
    }
}

