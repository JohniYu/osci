pipeline {
    agent none

    environment {
        IMAGE_NAME = 'resume-app'
        REGISTRY = 'asia-northeast3-docker.pkg.dev/crafty-isotope-455015-b6/resume-app'
        FULL_IMAGE = "${REGISTRY}/${IMAGE_NAME}"
        CONTAINER_NAME = 'resume-container'
        COMPOSE_PATH = '/data/osci/docker-compose.yml'
    }

    stages {
        stage('Build & Push') {
            agent { label 'build_agent' }
            environment {
                VERSION = "v-${env.BUILD_NUMBER}"
            }
            steps {
                git branch: 'main', url: 'https://github.com/JohniYu/osci.git'
                sh """
                    gcloud auth configure-docker asia-northeast3-docker.pkg.dev
                    docker build -t ${FULL_IMAGE}:${VERSION} .
                    docker tag ${FULL_IMAGE}:${VERSION} ${FULL_IMAGE}:latest

                    gcloud auth activate-service-account --key-file=/home/jenkins/key.json

                    docker push ${FULL_IMAGE}:${VERSION}
                    docker push ${FULL_IMAGE}:latest
                """
            }
        }

        stage('Deploy') {
            agent { label 'app_agent' }
            environment {
                VERSION = "v-${env.BUILD_NUMBER}"
            }
            steps {
                sh """
                    gcloud auth configure-docker asia-northeast3-docker.pkg.dev
                    docker compose -f ${COMPOSE_PATH} up -d --remove-orphans --force-recreate
                """
            }
        }
    }

    post {
        success {
            echo "빌드, 배포 성공"
        }
        failure {
            echo "오류 발생"
        }
    }
}
