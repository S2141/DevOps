pipeline {
    agent any

    environment {
        NODEJS_HOME = tool name: 'NodeJS 14.x', type: 'NodeJSInstallation'
        PATH = "${NODEJS_HOME}/bin:${env.PATH}"
        AWS_CREDENTIALS = credentials('aws-credentials-id') // Set up AWS credentials in Jenkins
        EC2_IP = '44.199.254.67' // Replace with your EC2 instance's public IP
        EC2_USER = 'ec2-user' // Replace with your EC2 username (typically 'ec2-user' for Amazon Linux)
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/S2141/DevOps.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'npm test'
            }
        }

        stage('Build Application') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Archive Artifacts') {
            steps {
                archiveArtifacts artifacts: 'out/**', fingerprint: true
            }
        }

        stage('Deploy to AWS EC2') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials-id']]) {
                    sh """
                    scp -o StrictHostKeyChecking=no -r out/* ${EC2_USER}@${EC2_IP}:/var/www/your-app/
                    ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_IP} 'sudo systemctl restart nginx'
                    """
                }
            }
        }

        stage('Test Deployment') {
            steps {
                sh "curl http://${EC2_IP}"
            }
        }
    }

    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Deployment failed!'
        }
    }
}
