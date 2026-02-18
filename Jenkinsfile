pipeline {
    agent any

    stages {

        stage('Build Backend Image') {
            steps {
                sh '''
                docker build -t backend-app backend
                '''
            }
        }

        stage('Deploy Backends') {
            steps {
                sh '''
                docker rm -f backend1 backend2 || true

                docker run -d --name backend1 --network bridge backend-app
                docker run -d --name backend2 --network bridge backend-app

                sleep 3
                '''
            }
        }

        stage('Configure Network and Run NGINX') {
            steps {
                sh '''
                docker rm -f nginx-lb || true

                docker network create mynet || true

                docker network connect mynet backend1 || true
                docker network connect mynet backend2 || true

                docker run -d -p 80:80 --name nginx-lb --network mynet nginx

                sleep 3

                docker cp nginx/default.conf nginx-lb:/etc/nginx/conf.d/default.conf

                docker exec nginx-lb nginx -s reload
                '''
            }
        }

    }

    post {
        success {
            echo 'Pipeline executed successfully. Load balancer is running.'
        }
        failure {
            echo 'Pipeline failed. Check console output.'
        }
    }
}