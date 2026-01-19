pipeline {
    agent any
    environment {
        ANSIBLE_SERVER_IP = '137.184.175.128'
    }
    stages {
        stage('Copy files to ansible server') {
            steps {
                echo 'Copying files to ansible control node...'
                sshagent(credentials: ['anisble-server-key']) {
                    sh '''
                        scp -o StrictHostKeyChecking=no -r ./ansible/* root@${ANSIBLE_SERVER_IP}:/root
                    '''
                    withCredentials([sshUserPrivateKey(credentialsId: 'anisble-server-key', keyFileVariable: 'KEYFILE', usernameVariable: 'USERNAME')]) {
                        sh '''
                            scp -o StrictHostKeyChecking=no  ${KEYFILE} root@${ANSIBLE_SERVER_IP}:/root/id_ed25519
                            '''
                    }
                }
            }
        }
        stage('Execute Ansible Playbook on Ansible Server') {
            steps {
                echo 'Calling Ansible Playbook to configure EC2 instances...'
                script {
                    def remote = [:]
                    remote.name = 'ansible-server'
                    remote.host = ANSIBLE_SERVER_IP
                    remote.allowAnyHosts = true

                    withCredentials([
                        sshUserPrivateKey(credentialsId: 'anisble-server-key', keyFileVariable: 'KEYFILE', usernameVariable: 'USERNAME'),

                    ]) {
                        remote.user = USERNAME
                        remote.identityFile = KEYFILE
                        sshCommand remote: remote, command: """
                            ansible-playbook my-playbook.yaml
                        """
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying...'
            }
        }
    }
}
