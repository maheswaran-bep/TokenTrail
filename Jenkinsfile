
pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
        timeout(time: 30, unit: 'MINUTES')
    }

    environment {
        COMPOSE_PROJECT_NAME = 'tokentrail'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Source code checked out by Jenkins SCM.'
            }
        }

        stage('Prepare Environment') {
            steps {
                withCredentials([
                    file(
                        credentialsId: 'tokentrail-env',
                        variable: 'ENV_FILE'
                    )
                ]) {
                    sh '''
                        set -eu
                        cp "$ENV_FILE" .env
                        chmod 600 .env
                        test -s .env
                        echo "Environment file prepared."
                    '''
                }
            }
        }

        stage('Validate Docker Compose') {
            steps {
                sh '''
                    set -eu
                    docker compose config --quiet
                    echo "Docker Compose configuration is valid."
                '''
            }
        }

        stage('Build Images') {
            steps {
                sh '''
                    set -eu
                    docker compose build
                '''
            }
        }

        stage('Deploy Application') {
            steps {
                sh '''
                    set -eu
                    docker compose up -d --build --wait --wait-timeout 180
                    docker compose ps
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    set -eu

                    echo "Checking frontend health..."
                    curl --retry 5 --retry-delay 3 --retry-connrefused \
                        -fsS http://host.docker.internal:3000/healthz

                    echo "Checking backend health..."
                    curl --retry 5 --retry-delay 3 --retry-connrefused \
                        -fsS http://host.docker.internal:8000/healthz

                    echo "Checking API through frontend..."
                    curl --retry 5 --retry-delay 3 --retry-connrefused \
                        -fsS http://host.docker.internal:3000/api/overview

                    echo "TokenTrail deployment verification successful."
                '''
            }
        }
    }

    post {
        always {
            sh 'rm -f .env'
            echo 'Pipeline completed. Temporary environment file removed.'
        }

        success {
            echo 'TokenTrail CI/CD pipeline completed successfully.'
        }

        failure {
            echo 'TokenTrail CI/CD pipeline failed. Check the stage logs.'
        }
    }
}
