pipeline {
    agent any
    environment {
        SONAR_URL = 'http://192.168.0.113:9000'
        NEXUS_URL = 'http://192.168.0.113:8081'
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
                    sh 'docker build -t 192.168.0.113:8082/dockerhosted-repo:latest .'
                }
            }
        }

stage('Push Docker image to docker hosted rerpository on Nexus') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'nexus', passwordVariable: 'PSW', usernameVariable: 'USER')]){
                    sh "echo ${PSW} | docker login -u ${USER} --password-stdin 192.168.0.113:8082/repository/dockerhosted-repo/"
                    sh "docker push 192.168.0.113:8082/dockerhosted-repo:latest"
                     }
                }
            }
        }
	    
  stage('Deploy to Local Kubernetes') {
            steps {
                script {
		    kubeconfig(caCertificate: 'C:\\Users\\emb-shaitan\\Downloads\\ca.crt', credentialsId: 'kubeconfig', serverUrl: 'https://192.168.49.2:8443') {
                    sh 'kubectl apply -f minikube-deployment.yaml'
                           } 
		 }
		}
	    }
        }
}
