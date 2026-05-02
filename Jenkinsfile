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
            -Dsonar.token=sqa_27b76609fd16a402abaee77c09c80fa15d26bcf6 \
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


        stage('Trivy Scan') {
    steps {
        sh '''
        export PATH=$PATH:/usr/local/bin
        trivy image --severity HIGH,CRITICAL --exit-code 1 ci-cd-app
        '''
    }
}
        


        stage('Push to DockerHub') {
    steps {
        withCredentials([usernamePassword(
            credentialsId: 'dockerhub-creds',
            usernameVariable: 'DOCKER_USER',
            passwordVariable: 'DOCKER_PASS'
        )]) {
            sh '''
            echo "$DOCKER_PASS" | sudo docker login -u "$DOCKER_USER" --password-stdin
            sudo docker tag ci-cd-app $DOCKER_USER/ci-cd-app:latest
            sudo docker push $DOCKER_USER/ci-cd-app:latest
            '''
        }
    }
}

        

        stage('Run Docker Container') {
    steps {
        withCredentials([usernamePassword(
            credentialsId: 'dockerhub-creds',
            usernameVariable: 'DOCKER_USER',
            passwordVariable: 'DOCKER_PASS'
        )]) {
            sh '''
            sudo docker stop ci-cd-app || true
            sudo docker rm ci-cd-app || true
            sudo docker pull $DOCKER_USER/ci-cd-app:latest
            sudo docker run -d -p 8082:8080 --name ci-cd-app $DOCKER_USER/ci-cd-app:latest
            '''
        }
      }
     }
        
    }
}
