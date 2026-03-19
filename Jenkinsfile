pipeline {
    agent any
    
    environment {
        // Standard Docker Compose command
        DOCKER_COMPOSE_CMD = 'docker compose'
    }

    stages {
        stage('Checkout Code') {
            steps {
                // Jenkins pulls the latest code from the GitHub branch you configure
                checkout scm
            }
        }
        
        stage('Inject Environment Variables') {
            steps {
                /*
                 * PRO-TIP FOR TOMORROW: 
                 * In a real production app, you NEVER store .env files in Git or hardcode them here.
                 * You securely store them in Jenkins using the "Credentials" page as "Secret Files".
                 * This code tells Jenkins to perfectly inject those hidden files into your workspace right before building!
                 */
                withCredentials([
                    file(credentialsId: 'client-env-secret', variable: 'CLIENT_ENV'),
                    file(credentialsId: 'server-env-secret', variable: 'SERVER_ENV')
                ]) {
                    sh 'cp $CLIENT_ENV client/.env'
                    sh 'cp $SERVER_ENV server/.env'
                }
            }
        }
        
        stage('Build Docker Containers') {
            steps {
                /* 
                 * Since your Dockerfiles already run `npm install` and `npm run build` 
                 * internally, this step automatically verifies that both your React 
                 * frontend and Node backend can compile flawlessly without needing 
                 * Node or NPM installed directly on the Jenkins server!
                 */
                sh '${DOCKER_COMPOSE_CMD} build'
            }
        }

        stage('Deploy to Server') {
            steps {
                /* 
                 * Scenario A: Jenkins is installed ON your EC2 server.
                 * It simply stops the old containers and starts the new ones.
                 */
                sh '${DOCKER_COMPOSE_CMD} down'
                sh '${DOCKER_COMPOSE_CMD} up -d'
                
                /*
                 * Scenario B: Jenkins is installed somewhere ELSE (like Jenkins Cloud).
                 * You would uncomment this block to SSH into your EC2 and deploy remotely instead:
                 * 
                 * sshagent(['ec2-ssh-credentials-id-in-jenkins']) {
                 *     sh 'ssh -o StrictHostKeyChecking=no ubuntu@13.127.128.51 "cd ~/thumblify_fullstack && git pull origin main && docker compose up -d --build"'
                 * }
                 */
            }
        }
    }
    
    post {
        always {
            // Cleans up the Jenkins workspace so it doesn't take up disk space
            cleanWs()
        }
        success {
            echo 'Deployment was a total success! The new version is live.'
        }
        failure {
            echo 'Pipeline failed. Quick, check the Jenkins console logs!'
        }
    }
}
