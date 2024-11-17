pipeline {
    agent any
    
    
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: '<repo-url>'
            }
        }
        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Trivy FS Scan') {
            steps {
                sh "trivy fs --format table -o fs.html ."
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube-server') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=<project-name> -Dsonar.projectKey=<project-key> -Dsonar.java.binaries=target'''
                }
            }
        }
        stage('Build') {
            steps {
                sh 'mvn package'
            }
        }
        stage('Publish Artifacts') {
            steps {
                withMaven(globalMavenSettingsConfig: '<global-settings-name>', jdk: '', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh 'mvn deploy'
                }
                
            }
        }
        stage('Docker Build and Tag Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: '<your-credentials>', toolName: 'docker') {
                        sh "docker build -t <dockerhub-userid>/<image_name>:<tag> ."
                    }
                }
            }
        }
        stage('Trivy Image Scan') {
            steps {
                sh "trivy image --format table -o fs.html <dockerhub-userid>/<image_name>:<tag>"
            }
        }
        stage('Push Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: '<your-credentials>', toolName: 'docker') {
                        sh "docker push <dockerhub-userid>/<image_name>:<tag>"
                    }
                }
            }
        }
        stage('Kubernetes Deploy'){
            steps{
                withKubeConfig(caCertificate:'',clusterName:'<cluster-name>',contextName:'',credentiaslID:'<your-credentiasl>',namespace:'<namespace>',restrictKubeConfigAccess:false,serverUrl:'<URL>'){
                    sh "kubectl apply -f deployment-service.yml"
                }
            }
        }
        stage('Verify Deployment'){
            steps{
                withKubeConfig(caCertificate:'',clusterName:'<cluster-name>',contextName:'',credentiaslID:'<your-credentiasl>',namespace:'<namespace>',restrictKubeConfigAccess:false,serverUrl:'<URL>'){
                    sh "kubectl get pods"
                    sh "kubectl get svc"
                }
            }
        }   
    }
    
}
