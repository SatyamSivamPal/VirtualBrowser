pipeline {
    agent any
    
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }

    stages {
        stage('Git-checkout') {
            steps {
               git branch: 'main' , url: 'https://github.com/SatyamSivamPal/VirtualBrowser.git'
            }
        }
         stage('Owasp Dependeency check') {
            steps {
                dependencyCheck additionalArguments: ' --scan ./ --disableNodeAudit', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
         stage('SonarQube') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=VirtualBrowser \
                    -Dsonar.projectName=VirtualBrowser '''
                }
            }
        }
         stage('Docker build and Tag') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'Docker') {
                        dir('/home/ubuntu/.jenkins/workspace/Virtual-Browser/.docker/firefox') {
                            sh "docker build -t satyam993/virtualbrowser:latest ."
                        }
                    }
                }
            }
        }
        stage('Trivy Scanner') {
            steps {
               sh "trivy image satyam993/virtualbrowser:latest > trivy.txt"
            }
        }
        stage('Docker Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'Docker') {
                        sh "docker push satyam993/virtualbrowser:latest"
                    }
                }
            }
        }
         stage('Deploy') {
            steps {
                sh "docker-compose up -d"
            }
        }
    }
    post {
        always {
            script {
                emailext attachLog: true,
                    subject: "${currentBuild.result}",
                    body: """
                        Project: ${env.JOB_NAME}
                        Build Number: ${env.BUILD_NUMBER}
                        URL: ${env.BUILD_URL}
                    """,
                    to: "satyamsivampal3@gmail.com",
                    attachmentsPattern: "trivy.txt"
            }
        }
    }
}
    





