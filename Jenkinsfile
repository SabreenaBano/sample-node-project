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
                    credentialsId: 'github-credentials'
            }
        }

        stage('Build') {
            steps {
                script {
                    // Build the Node.js application
                    sh 'cd /var/lib/jenkins/workspace/pipeline-2'
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
                    sshagent(['SSH_CREDENTIALS']) { // Updated to use the environment variable
                        // Copy the built application to EC2 using SCP
                        sh """
                        scp -o StrictHostKeyChecking=no -r dist/* ubuntu@${EC2_INSTANCE_IP}:/var/www/html/
                        """
                        // Restart the Node.js application on EC2
                        sh """
                        ssh -o StrictHostKeyChecking=no ubuntu@${EC2_INSTANCE_IP} \
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
        }
    }
}
