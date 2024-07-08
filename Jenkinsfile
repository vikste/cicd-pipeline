pipeline {
    agent any

    tools {
        nodejs 'NodeJS 22.4.0'
    }

    environment {
        NODE_ENV = 'development'
        DOCKER_IMAGE_MAIN = 'nodemain:v1.0'
        DOCKER_IMAGE_DEV = 'nodedev:v1.0'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'npm test'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    if (env.GIT_BRANCH == 'main') {
                        sh "docker build -t ${env.DOCKER_IMAGE_MAIN} ."
                    } else if (env.GIT_BRANCH == 'dev') {
                        sh "docker build -t ${env.DOCKER_IMAGE_DEV} ."
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    if (env.GIT_BRANCH == 'main') {
                        sh '''
                            CONTAINERS=$(docker ps -q --filter "ancestor=${env.DOCKER_IMAGE_MAIN}")
                            if [ -n "$CONTAINERS" ]; then
                                docker stop $CONTAINERS
                                docker rm $CONTAINERS
                            fi
                            docker run -d --expose 3000 -p 3000:3000 ${env.DOCKER_IMAGE_MAIN}
                        '''
                    } else if (env.GIT_BRANCH == 'dev') {
                        sh '''
                            CONTAINERS=$(docker ps -q --filter "ancestor=${env.DOCKER_IMAGE_DEV}")
                            if [ -n "$CONTAINERS" ]; then
                                docker stop $CONTAINERS
                                docker rm $CONTAINERS
                            fi
                            docker run -d --expose 3001 -p 3001:3000 ${env.DOCKER_IMAGE_DEV}
                        '''
                    }
                }
            }
        }
    }
}
