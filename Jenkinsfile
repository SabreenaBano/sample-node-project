pipeline {
    agent any

environment {
  // Use Jenkins credentials plugin to avoid exposing credentials directly
    GITHUB_CREDENTIALS = credentials('github-credentials') // GitHub credentials ID in Jenkins
    SSH_CREDENTIALS = credentials('ssh-credentials')    // EC2 SSH credentials ID in Jenkins
    EMAIL_CREDENTIALS = credentials('smtp-cred')  // Gmail credentials for notification
    EC2_INSTANCE_IP = '15.206.146.210'  // Updated EC2 instance's public IP
}

    stages {
        stage('Clone Code') {
            steps {
                // Clone the code from GitHub
                git branch: 'master', 
                    url: 'https://github.com/SabreenaBano/node-hello.git', 
                    credentialsId: 'github-credentials-id'
            }
        }

        stage('Build') {
            steps {
                script {
                    // Build the Node.js application
                    sh 'npm install'
                    sh 'npm run build'
                }
            }
        }

        stage('Approval') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    input message: 'Approve Deployment to EC2?', ok: 'Deploy'
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // Use SSH agent for EC2 credentials
                    sshagent(['ec2-ssh-credentials']) {
                        // Copy the built application to EC2 using SCP
                        sh """
                        scp -o StrictHostKeyChecking=no -r dist/* ubuntu@${15.206.146.210}:/var/www/html/
                        """
                        // Restart the Node.js application on EC2
                        sh """
                        ssh -o StrictHostKeyChecking=no ec2-user@${EC2_INSTANCE_IP} \
                            'pm2 restart all || sudo systemctl restart node-app.service'
                        """
                    }
                }
            }
        }
    }
    
    post {
        always {
            // Send email notification
            script {
                emailext (
                    subject: "Jenkins Job - Deployment Completed",
                    body: "The deployment of your Node.js application to EC2 instance ${EC2_INSTANCE_IP} is completed.",
                    to: 'sabreena.bano@shellkode.com',
                    replyTo: 'no-reply@example.com',
                    from: 'jenkins@example.com',
                    mimeType: 'text/plain',
                    recipientProviders: [[$class: 'DevelopersRecipientProvider']],
                    attachLog: true
                )
            }
            
            // Clean up workspace after the pipeline execution
            cleanWs()
        }
    }
}
