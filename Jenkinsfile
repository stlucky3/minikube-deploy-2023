pipeline {
    agent any
    environment {
        SONAR_URL = 'http://192.168.0.113:9000'
        NEXUS_URL = 'http://192.168.0.113:8081'
	KUBE_CONFIG = credentials('kubernetes-config')
        ECR_REPO = '981687212131.dkr.ecr.us-east-1.amazonaws.com' 
        REPO_NAME = 'embitel-ecr'
	    region = 'us-east-1'
	EKS_CLUSTER = 'embitel-EKS-cluster2'
    }
    stages {
        stage('Checkout From GitHub Repository') {
            steps {
                checkout scm
            }
        }
         stage('Maven Tool Build and Test Code') {
            steps {
                script {
                    sh 'mvn clean install'
                }
            }
        }
        stage('SonarQube Code Analysis') {
            steps {
                script {
                    withSonarQubeEnv('SonarQube') {
                        sh 'mvn sonar:sonar -Dsonar.host.url=$SONAR_URL'
                           
                    }
                }
            }
        }
         stage('Deploy to Nexus Repository') {
            steps {
                nexusArtifactUploader artifacts: [[artifactId: 'spring-boot-starter-parent', classifier: '', file: 'target/spring-boot-web.jar', type: 'jar']], credentialsId: 'Nexus-Maven', groupId: 'org.springframework.boot', nexusUrl: '192.168.0.113:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'embitel-maven', version: '1.0'
            }
        }
       stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t stlucky3/devops:latest .'
                }
            }
        }
           
  stage('Deploy') {
            steps {
                // Deploy to Kubernetes
                script {
                    withKubeConfig([credentialsId: 'kubernetes-config']) {
                        sh 'kubectl apply -f minikube-deployment.yaml' // Apply your Kubernetes manifests
                    }
                }
            }
        }
            }
}
