pipeline {
    parameters {
        string(name: 'BUILD_NODE', defaultValue: 'p4-dev', description: 'Build Machine')
    }
    agent {
        agent {
            label "${BUILD_NODE}"
        }
        docker { 
            image 'ubuntu:18.04' 
            args '-u root:sudo'
        }
    }
    stages {
        stage('Install tools') {
            steps {
                sh '''
                set -x
                ls
                cat Jenkinsfile
                '''
            }
        }
        stage('Publish') {
            when {
               branch 'master'
            }
            steps {
                sh '''
                set -x
                echo 'publish'
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

