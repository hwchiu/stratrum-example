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
                apt-get install -y curl wget
                
                # Install kubectl
                curl -LO "https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubectl"
                chmod +x ./kubectl
                mv ./kubectl /usr/local/bin/kubectl
                    
                # Install rancher    
                wget https://github.com/rancher/cli/releases/download/v2.4.5/rancher-linux-amd64-v2.4.5.tar.gz
                tar -xvf rancher-linux-amd64-v2.4.5.tar.gz
                mv rancher-v2.4.5/rancher /usr/local/bin
                
                # Test Kubectl & Rancher
                KUBE_CONFIG=$KUBECONFIG kubectl get nodes
                rancher --version
                '''
            }
        }
        stage('Login Rancher') {
            steps {
                sh '''
                rancher login ${rancher_server} --token ${rancher_token} --context ${rancher_context}:${rancher_project}
                '''
             }
        }
        stage('Uninstall Apps') {
            steps {
                sh '''
                apps=$(rancher apps -q)
                for app in $apps; do rancher apps delete $app; done
                '''
             }
        }  
        stage('Remove PVC') {
            steps {
                sh '''
                pvcs=$(kubectl -n onos-tost get pvc -lapp=onos-tost-atomix -o name)
                for pv in $pvcs; do kubectl -n onos-tost delete $pv; done
                '''
             }
        }    
        stage('Install apps') {
            steps {
                sh '''
                rancher apps install --namespace stratum ${rancher_context}:stratum-stratum stratum
                rancher apps install --namespace onos-tost ${rancher_context}:onos-onos-tost onos-tost
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
