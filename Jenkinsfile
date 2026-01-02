pipeline {
    agent any

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                    sonar-scanner \
                    -Dsonar.projectKey=static-website \
                    -Dsonar.projectName=static-website \
                    -Dsonar.sources=.
                    '''
                }
            }
        }

        stage('Build & Deploy on Docker Server') {
            steps {
                sh '''
                ssh -i /var/lib/jenkins/.ssh/docker_key \
                -o StrictHostKeyChecking=no \
                ubuntu@<DOCKER_PUBLIC_IP> "
                rm -rf ~/static-site &&
                git clone ${GIT_URL} static-site &&
                cd static-site &&
                docker build -t static-site:latest . &&
                docker rm -f css-website || true &&
                docker run -d -p 80:80 --name css-website static-site:latest
                "
                '''
            }
        }
    }
}
