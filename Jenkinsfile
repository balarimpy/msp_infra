pipeline {
    agent any
    
    tools{
        jdk  'jdk17'
        maven  'maven3'
    }
    
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
        DOCKER_BUILDKIT = '1'  // Enable Docker BuildKit
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
        stage('Setup Docker Buildx') {
            steps {
                script {
                    // Ensure Buildx is initialized
                    sh 'docker buildx create --use || echo "Buildx builder already exists"'
                }
            }
        }
        
        stage('Build & Tag Docker Image') {
            steps {
                script {
                    withCredentials([[
                        $class: 'AmazonWebServicesCredentialsBinding',
                        credentialsId: 'aws-ecr'
                    ]]) {
                        // AWS CLI login to ECR is no longer needed as the credential helper is used
                        
                        // Build and tag the Docker image
                        sh "docker build -t msp-repo ."
                        sh "docker tag msp-repo:latest 416668258315.dkr.ecr.us-east-1.amazonaws.com/msp-repo:latest"
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
                    withCredentials([[
                        $class: 'AmazonWebServicesCredentialsBinding',
                        credentialsId: 'aws-ecr'
                    ]]) {
                        // Push the Docker image to AWS ECR
                        sh "docker push 416668258315.dkr.ecr.us-east-1.amazonaws.com/msp-repo:latest"
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
    
        
    
    