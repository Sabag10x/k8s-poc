pipeline {
    agent any
    
    tools{
        jdk 'jdk17'
        maven 'maven3'
    }
    
    environment{
        SCANNER_HOME= tool 'sonar-scanner'
        PATH = "$PATH:/usr/local/bin/aws"
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/Sabag10x/k8s-poc.git'
            }
        }
        
        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }
        
        stage('Unit - Test') {
            steps {
                sh 'mvn test'
            }
        }
        
        stage('Sonarqube - Scanner') {
            steps {
                withSonarQubeEnv('sonar'){
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.ProjectName=K8sPoc -Dsonar.ProjectKey=K8sPoc \
                    -Dsonar.java.binaries=. '''
                    
                }
            }
        }
        
        stage('Sonar-Quality-Gate') {
            steps {
                script{
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn clean install'
            }
        }
        
        stage('Store Artifacts in S3') {
            steps {
                script {
                    withAWS(credentials: 's3-artifact', region: 'us-east-1') {
                        sh 'aws s3 cp ./target/*.jar  s3://poc-artifacts-ipsy/Artifacts/'
                    }
                }
            }
        }    
        
 //       stage('Publish-Artifacts -Nexus') {
//          steps {
//            withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
//                        sh 'mvn deploy'           
//                }
//            }
//        }
        
        stage('Build & Tag Docker Image ') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                            sh 'docker build -t sabag10x/k8-spoc:latest .'
                    }
                }
            }
        }
        
        stage('Docker-Image-Scan') {
            steps {
                sh "trivy image --format table -o trivy-fs-report.html sabag10x/k8-spoc:latest"
                
            }
        } 
        
        stage('Docker-Push Image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                            sh 'docker push sabag10x/k8-spoc:latest'
                    }
                }
            }
        }
        
    }
}
