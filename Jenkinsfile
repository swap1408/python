pipeline {
    agent any

    environment {
        REGION = "ap-south-1"
        AMI_ID = "ami-05b42df31a2765e57"
        INSTANCE_TYPE = "t3.micro"
        SUBNET_ID = "subnet-01fd350451bebd0ef"
        SG_ID = "sg-0ed03c6345f3d8f19"
        KEY_NAME = "mumbai"
        SCRIPT_FILE = "app.py"
    }

    stages {
        stage('Launch EC2 Instance') {
            steps {
                script {
                    INSTANCE_ID = sh(
                        script: """
                          aws ec2 run-instances --image-id ${AMI_ID} --count 1 --instance-type ${INSTANCE_TYPE} \
                          --key-name ${KEY_NAME} --subnet-id ${SUBNET_ID} --security-group-ids ${SG_ID} \
                          --region ${REGION} --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=jenkins-private}]' \
                          --query 'Instances[0].InstanceId' --output text
                        """, returnStdout: true
                    ).trim()
                    echo "✅ Launched Instance: ${INSTANCE_ID}"
                }
            }
        }

        stage('Get Private IP') {
            steps {
                script {
                    PRIVATE_IP = sh(
                        script: """
                          aws ec2 describe-instances --instance-ids ${INSTANCE_ID} --region ${REGION} \
                          --query 'Reservations[0].Instances[0].PrivateIpAddress' --output text
                        """, returnStdout: true
                    ).trim()
                    echo "📡 Private IP: ${PRIVATE_IP}"
                }
            }
        }

        stage('Wait for EC2') {
            steps {
                echo "⏳ Waiting 60 seconds for EC2 to boot up..."
                sh 'sleep 60'
            }
        }

        stage('Copy Script') {
            steps {
                echo "📁 Copying ${SCRIPT_FILE} to EC2"
                sshagent(['ec2-key']) {
                    sh """
                      scp -o StrictHostKeyChecking=no ${SCRIPT_FILE} ubuntu@${PRIVATE_IP}:/home/ubuntu/
                    """
                }
            }
        }

        stage('Run Script') {
            steps {
                echo "🚀 Running script on EC2"
                sshagent(['ec2-key']) {
                    sh """
                      ssh -o StrictHostKeyChecking=no ubuntu@${PRIVATE_IP} 'python3 /home/ubuntu/${SCRIPT_FILE}'
                    """
                }
            }
        }
    }

    post {
        always {
            echo "✅ Pipeline completed"
        }
    }
}
