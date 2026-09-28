pipeline {

    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Verify Files') {
            steps {
                sh '''
                    set -eu

                    echo "Commit:"
                    git rev-parse --short HEAD

                    test -f docker-compose.yml
                    test -f Dockerfile
                    test -f nginx.conf
                    test -f backend/Dockerfile

                    echo "Required deployment files found."
                '''
            }
        }

        stage('Docker Check') {
            steps {
                sh '''
                    docker --version
                    docker compose version
                    docker info >/dev/null
                '''
            }
        }

        stage('Compose Validation') {
            steps {
                sh '''
                    test -f /opt/leadflow/.env

                    docker compose \
                      --env-file /opt/leadflow/.env \
                      config -q

                    echo "Compose configuration is valid."
                '''
            }
        }

        stage('Build') {
            steps {
                sh '''
                    docker compose \
                      --env-file /opt/leadflow/.env \
                      build
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    docker compose \
                      --env-file /opt/leadflow/.env \
                      up -d
                '''
            }
        }

        stage('Status') {
            steps {
                sh '''
                    docker compose \
                      --env-file /opt/leadflow/.env \
                      ps
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                    sleep 10

                    curl --fail \
                         --retry 10 \
                         --retry-delay 3 \
                         --retry-connrefused \
                         http://localhost/api/health

                    echo
                    echo "LeadFlow API is healthy."
                '''
            }
        }
    }

    post {

        success {
            echo 'LeadFlow deployed successfully.'
        }

        failure {
            echo 'Deployment failed. Showing diagnostics.'

            sh '''
                docker compose \
                  --env-file /opt/leadflow/.env \
                  ps || true

                docker compose \
                  --env-file /opt/leadflow/.env \
                  logs --tail=100 backend || true

                docker compose \
                  --env-file /opt/leadflow/.env \
                  logs --tail=100 db || true

                docker compose \
                  --env-file /opt/leadflow/.env \
                  logs --tail=100 frontend || true
            '''
        }
    }
}
