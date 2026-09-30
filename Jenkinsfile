pipeline {
    agent any

    environment {
        PROJECT_DIR = "/home/ubuntu/Simple-Calculator-App"
        VITE_API_URL = "http://16.112.62.242:8000"
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
                    echo "Jenkins Workspace: ${WORKSPACE}"
                    echo "Deployment Directory: ${PROJECT_DIR}"

                    sudo mkdir -p "${PROJECT_DIR}"
                    sudo chmod 755 /home/ubuntu

                    sudo rsync -av --delete \
                        --exclude='.git' \
                        --exclude='node_modules' \
                        --exclude='dist' \
                        "${WORKSPACE}/" "${PROJECT_DIR}/"

                    sudo chown -R jenkins:jenkins "${PROJECT_DIR}"

                    echo "Project copied successfully"
                    echo "===== Project Files ====="
                    ls -la "${PROJECT_DIR}"
                '''
            }
        }

        stage('Create Environment Files') {
            steps {
                sh '''
                    cd "${PROJECT_DIR}"

                    cat > .env <<EOF
POSTGRES_DB=calculator_db
POSTGRES_USER=calculator
POSTGRES_PASSWORD=calculator123
DATABASE_URL=postgresql://calculator:calculator123@postgres:5432/calculator_db
CORS_ORIGINS=http://16.112.62.242:3000
VITE_API_URL=${VITE_API_URL}
EOF

                    echo "Environment file created"

                    echo "===== .env ====="
                    cat .env
                '''
            }
        }

        stage('Stop Old Containers') {
            steps {
                sh '''
                    cd "${PROJECT_DIR}"

                    echo "Stopping old containers..."

                    docker compose down || true

                    echo "Removing unused Docker resources..."

                    docker system prune -af --volumes || true
                '''
            }
        }

        stage('Build Docker Images') {
            steps {
                sh '''
                    cd "${PROJECT_DIR}"

                    echo "===== Building Docker Images ====="

                    docker compose build --no-cache
                '''
            }
        }

        stage('Deploy Containers') {
            steps {
                sh '''
                    cd "${PROJECT_DIR}"

                    echo "===== Starting Containers ====="

                    docker compose up -d
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    echo "Waiting for services..."
                    sleep 20

                    cd "${PROJECT_DIR}"

                    echo "===== Docker Compose Status ====="
                    docker compose ps

                    echo "===== Running Containers ====="
                    docker ps

                    echo "===== Backend Health ====="
                    curl -f http://localhost:8000/docs > /dev/null

                    echo "Backend is healthy"

                    echo "===== Frontend Health ====="
                    curl -f http://localhost:3000 > /dev/null

                    echo "Frontend is healthy"

                    echo "======================================"
                    echo "Application deployed successfully!"
                    echo "======================================"
                '''
            }
        }
    }

    post {

        success {
            echo "SUCCESS: Calculator Web App deployed successfully!"
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
