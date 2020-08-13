pipeline {
    agent {
        docker { 
            image 'ubuntu:18.04' 
            args '-u root:sudo'
        }
    }
    stages {
        stage('Code Checkout') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']]
                ])
            }
        }
        stage('Install tools') {
            steps {
                sh '''
                set -x
                ls
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

