pipeline {
    agent any
    tools{
        jdk 'jdk17'
        maven 'maven3'
    }
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Mrunal00712/Boardgame-Java-.git'
            }
        }
        stage('Build') {
            steps {
                script {
                    // Check if 'mvnw' file exists, if not, use system Maven
                    if (fileExists('mvnw')) {
                        sh 'chmod +x mvnw'
                        sh './mvnw clean package'
                    } else {
                        sh 'mvn clean package'
                    }
                }
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh 'mvn sonar:sonar'
                }
            }
        }
        stage('Security Scan') {
            steps {
                sh 'trivy fs --format table -o trivy-fs-report.html .'
            }
        }
        stage('Docker Build') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh 'docker build -t mrunal0712/boardshack:latest .'
                    }
                }
            }
        }
        
        stage('Docker Image Scan') {
            steps {
                sh 'trivy image --format table -o trivy-image-report.html mrunal0712/javafullstack:latest'
            }
        }
        
        
        stage('Docker Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh 'docker push mrunal0712/boardshack:latest '
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.9.170:6443') {
                    sh 'kubectl apply -f deployment-service.yaml'
                }
            }
        }
        
        stage('Verify The Deployment') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.9.170:6443') {
                    sh 'kubectl get pods -n webapps'
                    sh 'kubectl get svc -n webapps'
                }
            }
        }
        
        
    }
   post {
    always {
        script {
            def jobName = env.JOB_NAME
            def buildNumber = env.BUILD_NUMBER
            def pipelineStatus = currentBuild.result ?: 'UNKNOWN'
            def bannerColor = pipelineStatus.toUpperCase() == 'SUCCESS' ? 'green' : 'red'

            def body = """
                <html>
                <body>
                <div style="border: 4px solid ${bannerColor}; padding: 10px;">
                <h2>${jobName} - Build ${buildNumber}</h2>
                <div style="background-color: ${bannerColor}; padding: 10px;">
                <h3 style="color: white;">Pipeline Status: ${pipelineStatus.toUpperCase()}</h3>
                </div>
                <p>Check the <a href="${BUILD_URL}">console output</a>.</p>
                </div>
                </body>
                </html>
            """

            emailext (
                subject: "${jobName} - Build ${buildNumber} - ${pipelineStatus.toUpperCase()}",
                body: body,
                to: 'mrunalugemuge@gmail.com',
                from: 'jenkins@example.com',
                replyTo: 'jenkins@example.com',
                mimeType: 'text/html',
                attachmentsPattern: 'trivy-image-report.html'
            )
        }
    }
}


}
