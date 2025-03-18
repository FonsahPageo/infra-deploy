pipeline {
    agent any

    environment {
        ANSIBLE_HOST_KEY_CHECKING = "False"
        //modify to suit your volume mount path
        PRIVATE_KEY_PATH = "/var/jenkins_home/.ssh/FansoPageo"
    }

    stages {
        stage('Pull JenkinsFile'){
            steps{
                script {
                    git branch: 'fonsah', url: 'https://github.com/Techstargate/INFRASTRUCTURE.git'
                    sh 'svn export --force . /var/jenkins_home/ansible_temp'
                    sh 'mv /var/jenkins_home/ansible_temp/configManagement /var/jenkins_home/ansible'
                    sh 'rm -rf /var/jenkins_home/ansible_temp'
                }
            }
        }

        stage('Syntax Check') {
            steps {
                script {
                     sh '''
                    chmod 600 /var/jenkins_home/.ssh/FansoPageo
                    ansible-playbook -i /var/jenkins_home/ansible/hosts/hosts.ini \
                    /var/jenkins_home/ansible/playbooks/ansible_tools_check.yaml \
                    --syntax-check --diff
                    '''
                }
            }
        }

        stage('Pre-check Software Installation') {
            steps {
                script {
                    sh '''
                    // chmod 600 /var/jenkins_home/.ssh/FansoPageo
                    ansible-playbook -i /var/jenkins_home/ansible/inventory/hosts.ini \
                    /var/jenkins_home/ansible/playbooks/ansible_tools_check.yaml --tags check --diff \
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
            agent { label 'test' }
            steps {
                withCredentials([string(credentialsId: 'github_token', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                        rm -rf INFRASTRUCTURE-DEPLOY
                        git clone https://github.com/Techstargate/INFRASTRUCTURE-DEPLOY
                        rm -rf configManagement/.git
                        cp -R configManagement/* INFRASTRUCTURE-DEPLOY/
                        rm -rf configManagement
                        cd INFRASTRUCTURE-DEPLOY
                        git config user.email "example@gmail.com"
                        git config user.name "user"
                        git add .
                        git commit -m "Infrastructure-Deploy"
                        git push https://user:$GITHUB_TOKEN@github.com/user/INFRASTRUCTURE-DEPLOY.git
                    '''
                }
            }
        }

        // Uncomment and modify as necessary for actual installation
        // stage('Install Software') {
        //     steps {
        //         script {
        //             sh '''
        //             // chmod 600 /var/jenkins_home/.ssh/FansoPageo
        //             ansible-playbook -i /var/jenkins_home/ansible/inventory/hosts.ini \
        //             /var/jenkins_home/ansible/playbooks/ansible_tools_check.yaml --tags install --diff \
        //             --private-key ${PRIVATE_KEY_PATH}
        //             '''
        //         }
        //     }
        // }
    }
}
