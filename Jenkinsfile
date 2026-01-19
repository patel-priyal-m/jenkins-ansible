pipeline{
    agent any

    stages {
        stage('Copy files to ansible server') {
            steps {
                echo 'Copying files to ansible control node...'
                sshagent (credentials: ['anisble-server-key']) {
                    sh '''
                        scp -o StrictHostKeyChecking=no -r ./ansible/* root@137.184.175.128:/root 
                    '''
                    withCredentials([sshUserPrivateKey(credentialsId: 'ec2-server-key', keyFileVariable: 'KEYFILE', usernameVariable: 'USERNAME')]) {
                        sh '''
                            scp -o StrictHostKeyChecking=no  ${KEYFILE} $USERNAME@137.184.175.128:/root/ssh-key.pem
                            '''
                    }
                }
            }
        }
        stage('Test') {
            steps {
                echo 'Testing...'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying...'
            }
        }
    }
}