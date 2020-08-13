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
                echo "Cleaned Up Workspace For Project QQ"
                """
            }
        }
        stage('Install tools') {
            steps {
                sh '''
                set -x
                echo "test"
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

