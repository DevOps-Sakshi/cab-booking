pipeline {
    agent any

    tools {
        maven 'Maven'
    }

    stages {

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t cabbooking-app .'
            }
        }

        stage('Run Container') {
            steps {
                sh 'docker stop cab-app || true'
                sh 'docker rm cab-app || true'
                sh 'docker run -d -p 9090:8080 --name cab-app cabbooking-app'
            }
        }
    }
}
