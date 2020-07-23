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
        stage('Clone Config Repo') {
            options {
                timeout(time: 10, unit: "SECONDS")
            }            
            steps {
                sh '''
                git clone https://${git_user}:${git_password}@${git_server}/${git_repo}
                cd ${git_repo}
                git fetch "https://${git_user}:${git_password}@${git_server}/a/${git_repo}" refs/changes/27/19827/6 && git checkout FETCH_HEAD
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
        stage('Push Secrets') {
            steps {
                sh '''
                
                rancher namespaces ls | grep ${onos_ns} || rancher namespaces create ${onos_ns}
                
                kubectl -n ${onos_ns} delete secret git-secret --ignore-not-found=true
                kubectl -n ${onos_ns} create secret generic git-secret --from-literal=username=${git_user} --from-literal=password=${git_password}
                
                kubectl -n ${onos_ns} delete secret onos-secret --ignore-not-found=true
                kubectl -n ${onos_ns} create secret generic onos-secret --from-literal=username=${onos_user} --from-literal=password=${onos_password}
                '''
            }
        }
        
        stage('Uninstall Apps') {
            options {
                timeout(time: 90, unit: "SECONDS")
            }            
            steps {
                sh '''
                for app in $(rancher apps ls -q | grep -E '(onos-tost|stratum|telegraf)'); do rancher apps delete $app; done
                
                until [ "$(rancher apps ls -q | grep -E '(onos-tost|stratum|telegraf)')" = "" ]; do echo "wait deleted apps"; rancher apps ls ; sleep 1; done
                '''
             }
        }  
        stage('Remove PVC') {
            options {
                timeout(time: 300, unit: "SECONDS")
            }            
            steps {
                sh '''
                pvcs=$(kubectl -n onos-tost get pvc -lapp=onos-tost-atomix -o name)
                for pv in $pvcs; do kubectl -n onos-tost delete $pv; done
                '''
             }
        }    
        stage('Install apps') {
            options {
                timeout(time: 600, unit: "SECONDS")
            }
            steps {
                sh '''
                #rancher apps install --answers ${git_repo}/deployment-configs/aether/apps/menlo-tost-dev/stratum-ans.yml  --namespace ${stratum_ns} ${rancher_context}:stratum-stratum stratum
                rancher apps install --answers ${git_repo}/deployment-configs/aether/apps/menlo-tost-dev/onos-ans.yml --namespace ${onos_ns} ${rancher_context}:onos-onos-tost onos-tost
                rancher apps install --answers ${git_repo}/deployment-configs/aether/apps/menlo-tost-dev/telegraf-ans.yml --namespace ${telegraf_ns} ${rancher_context}:influxdata-telegraf telegraf
                
                apps=$(rancher apps -q)
                for app in $apps; do rancher wait $app --timeout 360; rancher apps ls; done                
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

