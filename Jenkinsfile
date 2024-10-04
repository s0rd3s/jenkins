pipeline {
    agent any
    environment {
        DATE_TAG = new Date().format('ddMMyy') // Формат тега даты DDMMYY
        DOCKER_REGISTRY = 's0rd3s'
        GITHUB_REPO = 'github.com/s0rd3s/PlaysDev_DevOps_Trainee.git'
        TARGET_SERVER_IP = '3.253.172.166'
        TARGET_USER = 'ec2-user'  // Имя пользователя на целевом сервере
    }
    stages {
        stage('Clone or Pull repository') {
            steps {
                script {
                    // Использование GitHub Personal Access Token для клонирования репозитория
                    withCredentials([string(credentialsId: 'dfac13a1-247a-466d-af2b-96d13131e874', variable: 'GITHUB_TOKEN')]) {
                        if (fileExists('PlaysDev_DevOps_Trainee')) {
                            // Если директория существует, выполняем git pull
                            dir('PlaysDev_DevOps_Trainee') {
                                sh '''
                                    git pull origin task13-ec2
                                '''
                            }
                        } else {
                            // Если директория не существует, выполняем git clone
                            sh '''
                                git clone -b task13-ec2 https://$GITHUB_TOKEN@$GITHUB_REPO
                            '''
                        }
                    }
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                dir('PlaysDev_DevOps_Trainee/nginx') {
                    sh '''
                        docker build -t $DOCKER_REGISTRY/nginx:$DATE_TAG .
                    '''
                }
                dir('PlaysDev_DevOps_Trainee/apache') {
                    sh '''
                        docker build -t $DOCKER_REGISTRY/apache:$DATE_TAG .
                    '''
                }
            }
        }

        stage('Docker Login and Push') {
            steps {
                // Логин в Docker Hub с использованием кредов Jenkins
                withCredentials([usernamePassword(credentialsId: '83f2cd0f-e9a0-4f79-8e59-89107cc9b7cc', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh '''
                        docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
                        docker push $DOCKER_REGISTRY/nginx:$DATE_TAG
                        docker push $DOCKER_REGISTRY/apache:$DATE_TAG
                    '''
                }
            }
        }

        stage('Clean up local Docker images') {
            steps {
                sh '''
                    docker rmi $DOCKER_REGISTRY/nginx:$DATE_TAG || true
                    docker rmi $DOCKER_REGISTRY/apache:$DATE_TAG || true
                '''
            }
        }

        stage('Deploy to Target Server') {
            steps {
                // Использование SSH-ключа напрямую
                sh '''
                    ssh -i ~/.ssh/id_rsa -o StrictHostKeyChecking=no $TARGET_USER@$TARGET_SERVER_IP '
                        echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin &&
                        docker pull $DOCKER_REGISTRY/nginx:$DATE_TAG &&
                        docker pull $DOCKER_REGISTRY/apache:$DATE_TAG &&
                        docker stop nginx || true &&
                        docker stop apache || true &&
                        docker rm nginx || true &&
                        docker rm apache || true &&
                        docker network create mynetwork || true &&
                        docker run -d --name apache --network=mynetwork -p 8080:8080 $DOCKER_REGISTRY/apache:$DATE_TAG &&
                        docker run -d --name nginx --network=mynetwork -p 80:80 $DOCKER_REGISTRY/nginx:$DATE_TAG
                    '
                '''
            }
        }
    }
}
