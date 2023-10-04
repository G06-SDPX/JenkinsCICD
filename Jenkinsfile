pipeline {
    agent any
    stages {
        stage('Install Environment') {
            agent {
                label 'test'
            }
            steps {
                sh 'python3 -m venv env'
            }
        }
        stage('Activate Environment && Install Library') {
            agent {
                label 'test'
            }
            steps {
                sh '''#!/bin/bash
                 source env/bin/activate && pip install -r ./requirements.txt
                '''
            }
        }
        stage('Run Unittest') {
            agent {
                label 'test'
            }
            steps {
                sh '''#!/bin/bash
                 source env/bin/activate && python3 ./unit_test.py
                '''
            }
        }
        stage('Run Robot') {
            agent {
                label 'test'
            }
            steps {
                echo 'Create Container'
                sh 'docker-compose -f ./compose.yaml up -d --build'
                echo 'Runing Robot'
                sh 'robot ./test-calculate.robot'
            }
        }
        stage('Push Registry') {
            agent {
                label 'test'
            }
            steps {
                echo 'Logging'
                sh 'docker login registry.gitlab.com'
                echo 'Build Images'
                sh 'cd app && docker build -t registry.gitlab.com/unnop1.tham/jenkinscicdtesting .'
                echo 'Push Images'
                sh 'docker push registry.gitlab.com/unnop1.tham/jenkinscicdtesting'
            }
        }
        stage('Clean Workspace') {
            agent {
                label 'test'
            }
            steps {
                echo 'DownTime'
                sh 'docker-compose -f ./compose.yaml down'
                sh 'docker system prune -a -f'
            }
        }
        stage('Stop and Remove Docker Container') {
            agent {
                label 'pre-prod'
            }
            steps {
                script {
                    def containerId
                    containerId = sh(script: 'docker ps -q -f "ancestor=registry.gitlab.com/unnop1.tham/jenkinscicdtesting"', returnStatus: true).trim()
                    
                    if (containerId) {
                        echo "Stopping and removing Docker container with ID: ${containerId}"
                        sh "docker stop ${containerId}"
                        sh "docker rm ${containerId}"
                    } else {
                        echo "No running container found with the specified image."
                    }
                }
            }
        }
        stage('Test pull image from GitLab') {
            agent {
                label 'pre-prod'
            }
            steps {
                echo 'Pull Image from Gitlab'
                sh 'docker pull registry.gitlab.com/unnop1.tham/jenkinscicdtesting'
                echo 'Run Contrainer'
                sh 'docker run -d -p 5000:5000 registry.gitlab.com/unnop1.tham/jenkinscicdtesting'
            }
        }
    }
}