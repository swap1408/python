pipeline {
    agent any
    environment {
        AWS_DEFAULT_REGION = 'ap-south-1'
    }
    stages {
        stage('Launch EC2 Instance') {
            steps {
                script {
                    INSTANCE_ID = sh(script: '''
                        aws ec2 run-instances \
                            --image-id ami-05b42df31a2765e57 \
                            --count 1 \
                            --instance-type t3.micro \
                            --key-name mumbai \
                            --subnet-id subnet-01fd350451bebd0ef \
                            --security-group-ids sg-0ed03c6345f3d8f19 \
                            --region ap-south-1 \
                            --tag-specifications ResourceType=instance,Tags=[{Key=Name,Value=jenkins-private}] \
                            --query 'Instances[0].InstanceId' \
                            --output text
                    ''', returnStdout: true).trim()
                    echo "✅ Launched Instance: ${INSTANCE_ID}"
                }
            }
        }

        stage('Get Private IP') {
            steps {
                script {
                    PRIVATE_IP = sh(script: """
                        aws ec2 describe-instances \
                            --instance-ids ${INSTANCE_ID} \
                            --region ap-south-1 \
                            --query 'Reservations[0].Instances[0].PrivateIpAddress' \
                            --output text
                    """, returnStdout: true).trim()
                    echo "📡 Private IP: ${PRIVATE_IP}"
                }
            }
        }

        stage('Wait for EC2') {
            steps {
                echo '⏳ Waiting 60 seconds for EC2 to boot up...'
                sh 'sleep 60'
            }
        }

        stage('Copy Script') {
            steps {
                echo '📁 Copying app.py to EC2'
                sshagent (credentials: ['jenkins-key']) {
                    sh """
                        scp -o StrictHostKeyChecking=no -i /var/lib/jenkins/.ssh/mumbai.pem app.py ec2-user@${PRIVATE_IP}:/home/ec2-user/
                    """
                }
            }
        }

        stage('Run Script') {
            steps {
                echo '🚀 Running app.py on EC2'
                sshagent (credentials: ['jenkins-key']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no -i /var/lib/jenkins/.ssh/mumbai.pem ec2-user@${PRIVATE_IP} 'python3 /home/ec2-user/app.py'
                    """
                }
            }
        }
    }

    post {
        always {
            echo '✅ Pipeline completed'
        }
    }
}
