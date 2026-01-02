pipeline {
    agent any

    tools {
        // This ensures the tool is downloaded/prepared on the agent
        "hudson.plugins.sonar.SonarRunnerInstallation" 'SonarQube'
    }

    stages {
        stage('Checkout Code') {
            steps {
                // Cleans workspace and pulls fresh code
                checkout scm
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    // 1. Get the path where Jenkins unpacked the scanner
                    def scannerHome = tool 'SonarQube'
                    
                    // 2. Wrap the execution with SonarQube server credentials
                    withSonarQubeEnv('SonarQube') {
                        sh "${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=static-website \
                        -Dsonar.projectName=static-website \
                        -Dsonar.sources=."
                    }
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
                git clone https://github.com/Humnamaq70149086/Jenkins-Docker-SonarQube.git static-site &&
                cd static-site &&
                sudo docker build -t static-site:latest . &&
                sudo docker rm -f css-website || true &&
                sudo docker run -d -p 80:80 --name css-website static-site:latest
                "
                '''
            }
        }
    }
    
    post {
        always {
            echo 'Pipeline execution finished.'
        }
        failure {
            echo 'Pipeline failed. Check the logs for more details.'
        }
    }
}
