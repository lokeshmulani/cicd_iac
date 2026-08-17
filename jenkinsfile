pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                url: 'https://github.com/lokeshmulani/terraform-demo.git'
            }
        }


	stage('AWS Test') {
    		steps {
        		sh 'whoami'
        		sh 'echo $HOME'
        		sh 'ls -la ~/.aws || true'
        		sh 'aws sts get-caller-identity'
    		}
	}



        stage('Terraform Init') {
            steps {
                dir('terraform') {
                    sh 'terraform init'
                }
            }
        }

        stage('Terraform Validate') {
            steps {
                dir('terraform') {
                    sh 'terraform validate'
                }
            }
        }

        stage('Terraform Plan') {
            steps {
                dir('terraform') {
                    sh 'terraform plan'
                }
            }
        }

        stage('Terraform Apply') {
            steps {
                dir('terraform') {
                    sh 'terraform apply -auto-approve'
                }
            }
        }

        stage('Get Public IP') {
            steps {
                script {
                    env.SERVER_IP = sh(
                        script: "cd terraform && terraform output -raw public_ip",
                        returnStdout: true
                    ).trim()

                    echo "Server IP: ${env.SERVER_IP}"
                }
            }
        }

stage('Wait For EC2') {
    steps {
        sh 'sleep 60'
    }
}


stage('Deploy Application') {
    steps {
        withCredentials([sshUserPrivateKey(
            credentialsId: 'ec2-key',
            keyFileVariable: 'SSH_KEY'
        )]) {
            sh '''
            scp -o StrictHostKeyChecking=no -i $SSH_KEY app/index.html ec2-user@$SERVER_IP:/tmp/index.html

            ssh -o StrictHostKeyChecking=no -i $SSH_KEY ec2-user@$SERVER_IP "
                sudo cp /tmp/index.html /var/www/html/index.html
                sudo chown apache:apache /var/www/html/index.html
                sudo systemctl restart httpd
            "
            '''
        }
    }
}
 
}
}
