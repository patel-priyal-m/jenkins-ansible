pipeline {
    agent any

    stages {
        stage('Copy files to ansible server') {
            steps {
                echo 'Copying files to ansible control node...'
                sshagent(credentials: ['anisble-server-key']) {
                    sh '''
                        scp -o StrictHostKeyChecking=no -r ./ansible/* root@137.184.175.128:/root
                    '''
                    withCredentials([sshUserPrivateKey(credentialsId: 'ec2-server-key', keyFileVariable: 'KEYFILE', usernameVariable: 'USERNAME')]) {
                        sh '''
                            scp -o StrictHostKeyChecking=no  ${KEYFILE} root@137.184.175.128:/root/ssh-key.pem
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
                    remote.host = '137.184.175.128'
                    remote.allowAnyHosts = true

                    withCredentials([sshUserPrivateKey(credentialsId: 'anisble-server-key', keyFileVariable: 'KEYFILE', usernameVariable: 'USERNAME')]) {
                        remote.user = USERNAME
                        remote.identityFile = KEYFILE
                        sshCommand remote: remote, command: '''
                            ls -l
                        '''
                    // ansible-playbook /root/playbook.yml --private-key /root/ssh-key.pem -i /root/hosts.ini
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
