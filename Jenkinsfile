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
        
        stage('Verify Backend') {
            steps {
                dir('server') {
                    // Install dependencies to ensure package.json is valid
                    sh 'npm ci'
                }
            }
        }

        stage('Verify Frontend') {
            steps {
                dir('client') {
                    // Install dependencies and run Vite's ESLint to catch early errors
                    sh 'npm ci'
                    sh 'npm run lint'
                }
            }
        }

        stage('Build Docker Containers') {
            steps {
                // This validates that both Dockerfiles build without errors
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
