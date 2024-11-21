pipeline {
    agent any    
    tools {       
        nodejs 'node16'
    } 
    environment {       
        SCANNER_HOME= tool 'sonar-scanner'
    }
    stages {
        stage('Checkout from GITHUB') {
            steps {
               git branch: 'master', url: 'https://github.com/cloudezigns/2048-React-CICD.git'
            }
        }   
        stage('Sonarqube Analysis') {
            steps {
               withSonarQubeEnv('sonar-server') {
                   sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Game \
                   -Dsonar.projectKey=Game '''
               }
            }
        }      
        stage('Qaulity Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-cred'
                }
            }
        }      
        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }     
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }     
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }       
        stage('Docker Build & Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-creds', toolName: 'docker') {
                        sh 'docker build -t 2048 .'
                        sh 'docker tag 2048 umerakmal104/2048:latest'
                        sh 'docker push umerakmal104/2048:latest'
                    }
                }
            }
        }     
        stage('TRIVY IMAGE SCAN') {
            steps {
                sh 'trivy image --timeout 20m umerakmal104/2048:latest > trivyimagereport.txt'
            }
        }     
        stage('Deploy Image to Container') {
            steps {
                sh 'docker run -d --name 2048 -p 3000:3000 umerakmal104/2048:latest'
            }
        }   
        stage('Deploy To Kubernetes Cluster') {
            steps {
                script {
                    withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s-cred', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                        sh 'kubectl apply -f deployment.yaml'
                    }
                }
            }
        }
    }
}
