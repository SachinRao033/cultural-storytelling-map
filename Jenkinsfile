pipeline {

    agent any

    environment {
        PROJECT_DIR = "/home/ubuntu/cultural-storytelling-map"
        EC2_IP = "3.111.78.229"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Copy Project') {
            steps {
                sh '''
                    echo "======================================"
                    echo "Jenkins Workspace: ${WORKSPACE}"
                    echo "Deployment Directory: ${PROJECT_DIR}"
                    echo "======================================"

                    sudo mkdir -p "${PROJECT_DIR}"

                    sudo cp -r "WORKSPACE/.""{PROJECT_DIR}/"

                    sudo chown -R jenkins:jenkins "${PROJECT_DIR}"

                    echo "Project copied successfully"
                '''
            }
        }

        stage('Create Backend Environment') {
            steps {
                sh '''
                    cd "${PROJECT_DIR}"

                    cat > backend/.env <<EOF
GEMINI_API_KEY=
GEMINI_MODEL=gemini-flash-lite-latest
NARRIFY_AUTO_APPROVE=true
JWT_SECRET_KEY=45ae66b9610c1299d62b759ed06d8e6fd8d0dc3b3f632cf3dae0235d4ed7b857
FRONTEND_URL=http://${EC2_IP}:3000
EOF

                    echo "Backend environment created"
                '''
            }
        }

        stage('Stop Old Containers') {
            steps {
                sh '''
                    cd "${PROJECT_DIR}"

                    echo "Stopping old containers..."

                    docker compose down || true
                '''
            }
        }

        stage('Build Docker Images') {
            steps {
                sh '''
                    cd "${PROJECT_DIR}"

                    echo "Building Docker images..."

                    export EC2_IP="${EC2_IP}"

                    docker compose build
                '''
            }
        }

        stage('Deploy Containers') {
            steps {
                sh '''
                    cd "${PROJECT_DIR}"

                    echo "Starting application..."

                    export EC2_IP="${EC2_IP}"

                    docker compose up -d
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    cd "${PROJECT_DIR}"

                    echo "Waiting for services..."
                    sleep 15

                    echo "======================================"
                    echo "Docker Compose Status"
                    echo "======================================"

                    docker compose ps

                    echo "======================================"
                    echo "Running Containers"
                    echo "======================================"

                    docker ps

                    echo "======================================"
                    echo "Backend Health Check"
                    echo "======================================"

                    curl -f http://localhost:8000

                    echo ""
                    echo "Backend is healthy"

                    echo "======================================"
                    echo "Frontend Health Check"
                    echo "======================================"

                    curl -f http://localhost:3000

                    echo ""
                    echo "Frontend is healthy"

                    echo "======================================"
                    echo "Database Check"
                    echo "======================================"

                    docker exec cultural-backend ls -lh /data

                    echo "======================================"
                    echo "Application deployed successfully!"
                    echo "======================================"
                '''
            }
        }
    }

    post {

        success {
            echo "SUCCESS: Cultural Storytelling Map deployed successfully!"
            echo "Frontend: http://${EC2_IP}:3000"
            echo "Backend: http://${EC2_IP}:8000"
        }

        failure {
            echo "FAILED: Deployment failed. Check Jenkins console output."
        }

        always {
            sh '''
                sudo chown -R jenkins:jenkins "${PROJECT_DIR}" || true
                docker image prune -f || true
            '''
        }
    }
}
