pipeline {
    agent any
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', credentialsId: 'docker-cred', url: 'https://github.com/Pramod858/Multi-Tier-Flask-Application.git'
            }
        }
        
        stage('Trivy File System Scan') {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
		
        stage('SonarQube Analsyis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectName=Flask-Postgres \
                    -Dsonar.projectKey=Flask-Postgres '''
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        
        stage('Build & Tag Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                    sh "docker build -t pramod858/flaskapp:latest ."
                    }
                }
            }
        }
        
        stage('Trivy Docker Image Scan') {
            steps {
                sh "trivy image --format table -o trivy-image-report.html pramod858/flaskapp:latest "
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                    sh "docker push pramod858/flaskapp:latest"
                    }
                }
            }
        }
        
        stage('Deploy to EKS') {
            steps {
                script {
                    withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: 'demo-eks-cluster', contextName: '', credentialsId: 'K8S-Token', namespace: 'webapps', serverUrl: 'https://055F8FCA54C7B678648B20D83FB07599.gr7.us-east-1.eks.amazonaws.com']]) {
                        sh "kubectl apply -f manifest.yml -n webapps"
                        sleep 60
                    }
                }
            }
        }
        
        stage('Verify the Deployment in EKS') {
            steps {
                script {
                    withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: 'demo-eks-cluster', contextName: '', credentialsId: 'K8S-Token', namespace: 'webapps', serverUrl: 'https://055F8FCA54C7B678648B20D83FB07599.gr7.us-east-1.eks.amazonaws.com']]) {
                        sh "kubectl get svc -n webapps"
                    }
                }
            }
        }
        

		
    }
    
}
