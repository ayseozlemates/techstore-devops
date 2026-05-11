// Guncelleme: Eksik paketler (pytest-cov, wget, unzip) ve gercek SonarQube ayarları eklendi.
pipeline {
    agent any

    environment {
        DOCKER_IMAGE    = 'techstore-app'
        SONAR_HOST      = 'http://host.docker.internal:9000' // Jenkins Docker'dan dışarıdaki Sonar'a erişmek için
        SONAR_TOKEN     = 'admin'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo "✅ Kod GitHub'dan alındı"
            }
        }

        stage('Setup') {
            steps {
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                    pip install pytest-cov
                '''
                echo "✅ Python sanal ortamı ve kütüphaneler (pytest-cov dahil) hazır"
            }
        }

        stage('Unit Tests') {
            steps {
                sh '''
                    . venv/bin/activate
                    pytest tests/test_app.py -v --tb=short --junit-xml=test-results/unit-tests.xml --cov=app --cov-report=xml:coverage.xml || true
                '''
            }
            post {
                always {
                    junit testResults: 'test-results/unit-tests.xml', allowEmptyResults: true
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                sh '''
                    . venv/bin/activate
                    wget -qO- https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-5.0.1.3006-linux.zip > sonar.zip
                    unzip -q sonar.zip
                    ./sonar-scanner-5.0.1.3006-linux/bin/sonar-scanner \
                        -Dsonar.projectKey=techstore \
                        -Dsonar.projectName="TechStore E-Commerce" \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=${SONAR_HOST} \
                        -Dsonar.login=${SONAR_TOKEN}
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ${DOCKER_IMAGE}:${env.BUILD_NUMBER} -t ${DOCKER_IMAGE}:latest .
                """
                echo "✅ Docker imajı oluşturuldu"
            }
        }

        stage('Deploy') {
            steps {
                sh """
                    docker stop techstore-app 2>/dev/null || true
                    docker rm techstore-app 2>/dev/null || true
                    docker run -d --name techstore-app --restart unless-stopped -p 5000:5000 ${DOCKER_IMAGE}:latest
                """
                echo "✅ Uygulama ayağa kaldırıldı"
            }
        }
    }
}