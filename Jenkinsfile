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
                        
                        // Debug: Check AWS credentials
                        sshCommand remote: remote, command: """
                            echo "=== DEBUG: Checking AWS credentials ==="
                            env | grep AWS || echo "No AWS env vars found"
                            
                            echo "=== DEBUG: Checking AWS config files ==="
                            ls -la ~/.aws/ 2>/dev/null || echo "No ~/.aws directory"
                            
                            echo "=== DEBUG: Checking if boto3 is installed ==="
                            python3 -c "import boto3; print('boto3 version:', boto3.__version__)" || echo "boto3 not installed"
                            
                            echo "=== DEBUG: Testing AWS CLI ==="
                            aws ec2 describe-instances --region us-east-2 2>&1 || echo "AWS CLI failed"
                            
                            echo "=== DEBUG: Testing Ansible inventory ==="
                            ansible-inventory --list -i inventory_aws_ec2.yaml 2>&1
                            
                            echo "=== DEBUG: Running playbook ==="
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
