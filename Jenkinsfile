pipeline {
    agent any
    environment {
        DOCKER_IMAGE = 'genesisdada/flask-app'
        DOCKER_TAG   = "${BUILD_NUMBER}"
    }
    stages {
        stage('Code Fetch') {
            steps {
                echo 'Fetching latest code from GitHub...'
                git branch: 'master',
                    url: 'https://github.com/DaniyalAhmed-kh/two-tier-flask-app.git'
            }
        }
        stage('Docker Image Creation') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                    sh "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest"
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                        sh "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
                        sh "docker push ${DOCKER_IMAGE}:latest"
                    }
                }
            }
        }
        stage('Kubernetes Deployment') {
            steps {
                script {
                    sh 'kubectl apply -f k8s/mysql-secret.yml'
                    sh 'kubectl apply -f k8s/mysql-deployment.yml'
                    sh 'kubectl apply -f k8s/mysql-service.yml'
                    sh 'kubectl apply -f k8s/flask-deployment.yml'
                    sh 'kubectl apply -f k8s/flask-service.yml'
                    sh 'kubectl rollout status deployment/flaskapp --timeout=120s'
                    sh 'kubectl get pods'
                    sh 'kubectl get svc'
                }
            }
        }
        stage('Prometheus/Grafana Monitoring') {
            steps {
                script {
                    sh 'kubectl get pods -n monitoring'
                    sh 'kubectl apply -f k8s/flask-servicemonitor.yml'
                    sh 'kubectl get svc -n monitoring'
                }
            }
        }
    }
    post {
        success { echo 'Pipeline completed successfully!' }
        failure { echo 'Pipeline failed.' }
    }
}
