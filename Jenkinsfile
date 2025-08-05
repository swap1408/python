pipeline {
    agent any

    environment {
        AMI_ID = 'ami-05b42df31a2765e57'          // ✅ Use your region's AMI
        INSTANCE_TYPE = 't2.micro'
        KEY_NAME = 'mumbai.pem'                       // ✅ Your EC2 key pair name
        SUBNET_ID = 'subnet-01fd350451bebd0ef'             // ✅ Private subnet ID
        SECURITY_GROUP_ID = 'sg-0ed03c6345f3d8f19'         // ✅ Security group allowing SSH
        KEY_PATH = '/home/jenkins/.ssh/key.pem'   // ✅ .pem path on Jenkins
        USER = 'ubuntu'                         // 'ubuntu' for Ubuntu AMI
        SCRIPT = 'app.py'                         // Script you want to run
        REGION = 'ap-south-1'
    }

    stages {

        stage('Launch EC2 Instance') {
            steps {
                script {
                    def launchCmd = """
                        aws ec2 run-instances \
                          --image-id $AMI_ID \
                          --count 1 \
                          --instance-type $INSTANCE_TYPE \
                          --key-name $KEY_NAME \
                          --subnet-id $SUBNET_ID \
                          --security-group-ids $SECURITY_GROUP_ID \
                          --region $REGION \
                          --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=jenkins-private}]' \
                          --query 'Instances[0].InstanceId' \
                          --output text
                    """
                    env.INSTANCE_ID = sh(script: launchCmd, returnStdout: true).trim()
                    echo "✅ Launched Instance: ${env.INSTANCE_ID}"
                }
            }
        }

        stage('Get Private IP') {
            steps {
                script {
                    def getIp = """
                        aws ec2 describe-instances \
                          --instance-ids $INSTANCE_ID \
                          --region $REGION \
                          --query 'Reservations[0].Instances[0].PrivateIpAddress' \
                          --output text
                    """
                    env.PRIVATE_IP = sh(script: getIp, returnStdout: true).trim()
                    echo "📡 Private IP: ${env.PRIVATE_IP}"
                }
            }
        }

        stage('Wait for EC2') {
            steps {
                echo "⏳ Waiting 60 seconds for EC2 to boot up..."
                sh "sleep 60"
            }
        }

        stage('Copy Script') {
            steps {
                echo "📁 Copying $SCRIPT to EC2"
                sh """
                    scp -i $KEY_PATH -o StrictHostKeyChecking=no $SCRIPT $USER@$PRIVATE_IP:/home/$USER/
                """
            }
        }

        stage('Run Script') {
            steps {
                echo "🚀 Running script on EC2"
                sh """
                    ssh -i $KEY_PATH -o StrictHostKeyChecking=no $USER@$PRIVATE_IP 'python3 /home/$USER/$SCRIPT'
                """
            }
        }
    }

    post {
        always {
            echo "✅ Pipeline completed"
        }
    }
}
