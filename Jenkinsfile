pipeline {

    agent any

    environment {
        PROJECT_DIR = "${WORKSPACE}"
        EC2_IP = "3.111.78.229"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Verify Project') {
            steps {
                sh '''
                    echo "======================================"
                    echo "Checking project structure"
                    echo "======================================"

                    test -f backend/requirements.txt
                    test -f backend/Dockerfile
                    test -f frontend/package.json
                    test -f frontend/Dockerfile
                    test -f docker-compose.yml

                    echo "Project structure OK"
                '''
            }
        }

        stage('Create Backend Environment') {
            steps {
                sh '''
                    echo "======================================"
                    echo "Creating backend environment"
                    echo "======================================"

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

        stage('Stop Existing Containers') {
            steps {
                sh '''
                    echo "======================================"
                    echo "Stopping existing containers"
                    echo "======================================"

                    docker compose down || true

                    echo "Existing containers stopped"
                '''
            }
        }

        stage('Build Docker Images') {
            steps {
                sh '''
                    echo "======================================"
                    echo "Building Docker images"
                    echo "======================================"

                    export EC2_PUBLIC_IP="${EC2_IP}"

                    docker compose build --no-cache

                    echo "Docker images built successfully"
                '''
            }
        }

        stage('Start Application') {
            steps {
                sh '''
                    echo "======================================"
                    echo "Starting application"
                    echo "======================================"

                    export EC2_PUBLIC_IP="${EC2_IP}"

                    docker compose up -d

                    echo "Application containers started"
                '''
            }
        }

        stage('Wait for Application') {
            steps {
                sh '''
                    echo "Waiting for application to start..."

                    sleep 15

                    echo "Checking containers..."

                    docker compose ps
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                    echo "======================================"
                    echo "Running health checks"
                    echo "======================================"

                    echo "Checking backend..."

                    curl -f http://localhost:8000/

                    echo ""
                    echo "Backend is healthy"

                    echo "Checking frontend..."

                    curl -f http://localhost:3000/

                    echo ""
                    echo "Frontend is healthy"

                    echo "Checking database..."

                    docker exec cultural-backend ls -lh /data

                    echo "Database volume is available"

                    echo "======================================"
                    echo "APPLICATION IS HEALTHY"
                    echo "======================================"
                '''
            }
        }
    }

    post {

        success {
            echo "======================================"
            echo "CULTURAL STORYTELLING DEPLOYMENT SUCCESSFUL"
            echo "======================================"

            echo "Frontend: http://${EC2_IP}:3000"
            echo "Backend:  http://${EC2_IP}:8000"
            echo "Swagger:  http://${EC2_IP}:8000/docs"

            echo "======================================"
        }

        failure {
            echo "======================================"
            echo "DEPLOYMENT FAILED"
            echo "======================================"

            echo "Check the Jenkins Console Output for the error."

            echo "======================================"
        }

        always {
            sh '''
                echo "======================================"
                echo "FINAL CONTAINER STATUS"
                echo "======================================"

                docker ps

                echo "======================================"
                echo "DOCKER COMPOSE STATUS"
                echo "======================================"

                docker compose ps || true
            '''
        }
    }
}
