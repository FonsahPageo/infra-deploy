pipeline {
    agent any

    environment {
        ANSIBLE_HOST_KEY_CHECKING = "False"
        //modify to suit your volume mount path
        PRIVATE_KEY_PATH = "/var/jenkins_home/.ssh/FansoPageo"
    }

    stages {
        stage('Set SSH Key Permissions') {
            steps {
                script {
                    sh 'chmod 600 /var/jenkins_home/.ssh/FansoPageo'
                }
            }
        }

        stage('Pull JenkinsFile'){
            steps{
                script {
                    sh 'rm -rf infra'
                    git branch: 'main', url: 'https://github.com/FonsahPageo/infra.git'
                }
            }
        }

        stage('Export configManagement'){
            steps{
                script{
                    sh 'cp -R configManagement/* /var/jenkins_home/ansible'
                }
            }
        }

        stage('Syntax Check') {
            steps {
                script {
                     sh '''
                    ansible-playbook -i /var/jenkins_home/ansible/inventory/hosts.ini \
                    /var/jenkins_home/ansible/playbooks/tools_check.yaml \
                    --syntax-check --diff
                    '''
                }
            }
        }

        stage('Pre-check Software Installation') {
            steps {
                script {
                    sh '''
                    ansible-playbook -i /var/jenkins_home/ansible/inventory/hosts.ini \
                    /var/jenkins_home/ansible/playbooks/tools_check.yaml --diff \
                    --private-key ${PRIVATE_KEY_PATH}
                    '''
                }
            }
        }

        stage('Manual Approval') {
            steps {
                input message: 'Pre-check passed. Do you want to proceed with the software installation?'
            }
        }

        stage('Copy code to infra-deploy') {
            steps {
                withCredentials([string(credentialsId: 'github_token', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                        rm -rf infra-deploy
                        git clone https://github.com/FonsahPageo/infra-deploy.git
                        rm -rf configManagement/.git
                        cp -R configManagement/* infra-deploy/
                        rm -rf configManagement
                        cd infra-deploy
                        git config user.email "ashprincepageo@gmail.com"
                        git config user.name "FonsahPageo"
                        git add .
                        git commit -m "Infrastructure-Deploy"
                        git push https://FonsahPageo:$GITHUB_TOKEN@github.com/FonsahPageo/infra-deploy.git
                    '''
                }
            }
        }

        // Uncomment and modify as necessary for actual installation
        // stage('Install Software') {
        //     steps {
        //         script {
        //             sh '''
        //             ansible-playbook -i /var/jenkins_home/ansible/inventory/hosts.ini \
        //             /var/jenkins_home/ansible/playbooks/tools_check.yaml --diff \
        //             --private-key ${PRIVATE_KEY_PATH}
        //             '''
        //         }
        //     }
        // }
    }
}
