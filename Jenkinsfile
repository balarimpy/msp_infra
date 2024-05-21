pipeline {
    agent any
    
    tools{
        jdk  'jdk17'
        maven  'maven3'
    }
    
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/balarimpy/msp_infra'
            }
        }
        
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        
        stage('Unit Tests') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=MSP_DEMO -Dsonar.projectName=MSP_DEMO \
                    -Dsonar.java.binaries=. '''
                    
                }
            }
            
        }
        stage('OWASP DEpendency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Build') {
            steps {
                sh "mvn package -DskipTests=true"
            }
        }
        
        stage('Deploy To Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-maven', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy -DskipTests=true"
                }
            }
        }
        
        stage('Build & Tag Docker Image') {
            steps {
                script {
                
                    withDockerRegistry(credentialsId: 'aws-ecr') {
                        sh "aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 416668258315.dkr.ecr.us-east-1.amazonaws.com"
                        sh "docker build -t msp-repo ."
                        sh "docker tag msp-repo:latest 416668258315.dkr.ecr.us-east-1.amazonaws.com/msp-repo:latest"
                      //  sh "docker tag  msp_demo rimpybala/msp_demo:latest"
                        
                    }
                }
            }
        }
        
        stage('Trivy Scan') {
            steps {
                sh "trivy image 416668258315.dkr.ecr.us-east-1.amazonaws.com/msp-repo:latest > trivy-report.txt "
                
            }
        }
        
        stage('Push The Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-hub-cred', toolName: 'docker') {
                        sh "docker push 416668258315.dkr.ecr.us-east-1.amazonaws.com/msp-repo:latest"
                       // sh "docker push rimpybala/msp_demo:latest"
                    }
                }
                 
            }
        }
        
        // stage('Kubernetes Deploy') {
        //     steps {
        //         withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://192.168.122.144:6443') {
        //             sh "kubectl apply -f deploymentservice.yml -n webapps"
        //             sh "kubectl get svc -n webapps"
    
        //         }
        //     }
        // }   
        
    }
}
    
        
    
    