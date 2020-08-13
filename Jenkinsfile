pipeline {
    agent {
        docker { 
            image 'ubuntu:18.04' 
            args '-u root:sudo'
        }
    }
    environment {
        KUBECONFIG = credentials("hwchiu-dev-cluster")
    }
    stages {
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
    post { 
        always { 
            cleanWs()
        }
    }    
}

