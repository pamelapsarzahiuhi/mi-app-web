pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh 'docker build -t mi-app-web:${BUILD_NUMBER} .'
            }
        }

        stage('Security Scan') {
            steps {
                sh '''
                    docker run --rm \
                      -v /var/run/docker.sock:/var/run/docker.sock \
                      aquasec/trivy \
                      image --severity HIGH,CRITICAL \
                      --exit-code 1 \
                      mi-app-web:${BUILD_NUMBER}
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    docker rm -f app-web-test || true
                    docker run -d --name app-web-test mi-app-web:${BUILD_NUMBER}
                    sleep 3

                    APP_IP=$(docker inspect -f \
                      "{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}" \
                      app-web-test)

                    curl -f http://$APP_IP:3000/

                    docker rm -f app-web-test
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh 'docker rm -f app-web || true'
                sh 'docker run -d -p 8080:3000 --name app-web mi-app-web:${BUILD_NUMBER}'
            }
        }
    }
}
