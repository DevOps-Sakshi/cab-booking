pipeline {
    agent any

    tools {
        maven 'Maven'
        jdk 'java'
    }

    stages {

        stage('Clone Code') {
            steps {
                git branch: 'main', url: 'https://github.com/DevOps-Sakshi/cab-booking.git'
            }
        }


        stage('Build + SonarQube Analysis') {
    steps {
        withSonarQubeEnv('sonar') {
            sh '''
            export SONAR_SCANNER_OPTS="-Xmx2048m"
            mvn clean verify sonar:sonar \
            -Dsonar.projectKey=ci-cd \
            -Dsonar.projectName=CI-CD \
            -Dsonar.token=sqa_8fcf3f95d7df73ed2d1d964a4122ab4df75e6965 \
            -Dsonar.exclusions=taxi-booking/src/main/webapp/** \
            -Dsonar.javascript.detectBundles=false
            '''
        }
    }
}


        stage('Build with Maven') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'sudo docker build -t ci-cd-app .'
            }
        }

        stage('Run Docker Container') {
            steps {
                sh '''
                sudo docker stop ci-cd-app || true
                sudo docker rm ci-cd-app || true
                sudo docker run -d -p 8082:8080 --name ci-cd-app ci-cd-app
                '''
            }
        }
    }
}
