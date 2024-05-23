pipeline {
    agent any
    
    tools{
        jdk  'jdk17'
        maven  'maven3'
    }
    
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
        IMAGE_REPO_NAME = "msp-repo"
        IMAGE_TAG = "latest"
        REPOSITORY_URI = "416668258315.dkr.ecr.us-east-1.amazonaws.com/msp-repo"
        AWS_DEFAULT_REGION= "us-east-1"
        AWS_ACCOUNT_ID = "416668258315"
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
        stage('Logging into AWS ECR') {
            steps {
                script {
                sh "aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
                }
                 
            }
        }
    // Building Docker images
        stage('Building image') {
        steps{
            script {
            dockerImage = docker.build "${IMAGE_REPO_NAME}:${IMAGE_TAG}"
            }
        }
        }

        
        stage('Trivy Scan') {
            steps {
                sh "trivy image 416668258315.dkr.ecr.us-east-1.amazonaws.com/msp-repo:latest > trivy-report.txt "
                
            }
        }
        
        stage('Pushing to ECR') {
            steps{  
            script {
                    sh "docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}:$IMAGE_TAG"
                    sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${IMAGE_TAG}"
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
    
        
    
    