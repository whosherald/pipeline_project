pipeline {
    agent any

    environment {
        SSH_CREDENTIALS_ID = '' // Replace with your actual SSH credentials ID
        SERVER_IP = 'ec2-54-224-197-21.compute-1.amazonaws.com' // monitored (the server we are deploying to)
        REMOTE_USER = 'ubuntu' // Change thhttps://github.com/Mutuwa99/thato_pro.gitis to the appropriate non-root user
        GITHUB_REPO_URL = 'https://github.com/SthandwaKay/Jenkins-server.git'
    }

    stages {
        stage('Prepare') {
            steps {
                script {
                    // Clean workspace
                    deleteDir()

                    // Checkout the code from the GitHub repository
                    checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: GITHUB_REPO_URL]]])
                }
            }
        }

        stage('Test SSH Connection') {
            steps {
                script {
                    try {
                        withCredentials([file(credentialsId: SSH_CREDENTIALS_ID, variable: 'SSH_KEY')]) {
                            // Add the server's host key to known_hosts
                            sh "ssh-keyscan -H ${SERVER_IP} >> ~/.ssh/known_hosts"

                            // Test the connection
                            def sshCommand = "ssh -i \$SSH_KEY -o StrictHostKeyChecking=no ${REMOTE_USER}@${SERVER_IP} 'echo Connection successful'"
                            def sshResult = sh(script: sshCommand, returnStatus: true)

                            if (sshResult == 0) {
                                echo "SSH Connection to ${REMOTE_USER}@${SERVER_IP} successful."
                            } else {
                                error "SSH Connection failed. Status code: ${sshResult}"
                            }
                        }
                    } catch (Exception e) {
                        error "An error occurred while testing the SSH connection: ${e.message}"
                    }
                }
            }
        }


        stage('Deploy Static Website') {
            steps {
                script {
                    // Ensure remote directory exists
                   // sh "ssh -i \$SSH_KEY ${REMOTE_USER}@${SERVER_IP} 'mkdir -p /var/www/html/'"

                    // Deploy the code tohhhh the server using scp
                    withCredentials([file(credentialsId: SSH_CREDENTIALS_ID, variable: 'SSH_KEY')]) {
                        try {
                            sh "scp -i \$SSH_KEY -o StrictHostKeyChecking=no -r ./ ${REMOTE_USER}@${SERVER_IP}:/var/www/html/"
                            echo 'Deployment successful!'
                        } catch (Exception e) {
                            echo "Deployment failed: ${e.message}"
                            error 'Deployment failed!'
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline successful!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}