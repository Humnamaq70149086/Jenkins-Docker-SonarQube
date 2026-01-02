pipeline {
    agent any

    tools {
        // Use 'sonar-scanner' as the tool type. 
        // 'SonarQube' must match the name you gave it in Global Tool Configuration.
        "hudson.plugins.sonar.SonarRunnerInstallation" 'SonarQube'
    }

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
                ubuntu@13.220.172.128 "
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
