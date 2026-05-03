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
            -Dsonar.token=sqa_94da3ed68eb0050c327ee70f96b988df4dd81b98 \
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


        
        stage('Trivy Image Scan') {
    steps {
        sh '''
        /usr/local/bin/trivy image \
        --severity HIGH,CRITICAL \
        --exit-code 1 \
        --format table \
        -o trivy-report.txt \
        ci-cd-app
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
        
    }

    stage('Deploy using Ansible') {
    steps {
        sh 'ansible-playbook -i ansible/hosts ansible/deploy.yml'
    }
}

    post {
        always {
            archiveArtifacts artifacts: 'trivy-report.txt', fingerprint: true
        }
    }
}
