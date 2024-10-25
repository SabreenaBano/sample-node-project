pipeline {
    agent any

    environment {
        GITHUB_CREDENTIALS = credentials('github-credentials') // GitHub credentials ID in Jenkins
        SSH_CREDENTIALS = credentials('ssh-credentials')    // EC2 SSH credentials ID in Jenkins
        EMAIL_CREDENTIALS = credentials('smtp-cred')  // Gmail credentials for notification
        EC2_INSTANCE_IP = '15.206.146.210'  // Updated EC2 instance's public IP
    }

    stages {
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
            cleanWs()
        }
    }
  }
}