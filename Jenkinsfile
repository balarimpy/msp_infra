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
        stage('Setup Buildx') {
            steps {
                script {
                    // Download and setup Buildx
                    sh '''
                    mkdir -p ~/.docker/cli-plugins
                    wget https://github.com/docker/buildx/releases/latest/download/docker-buildx-linux-amd64 -O ~/.docker/cli-plugins/docker-buildx
                    chmod +x ~/.docker/cli-plugins/docker-buildx
                    docker buildx create --use
                    '''
                }
            }
        }

        stage('Building image and Tagging') {
            steps {
                script {
                    // Build and tag the Docker image using Buildx
                    sh """
                    docker buildx build --platform linux/amd64 -t ${IMAGE_REPO_NAME}:${IMAGE_TAG} .
                    docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}:${IMAGE_TAG}
                    """
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
                        sh """docker push 416668258315.dkr.ecr.us-east-1.amazonaws.com/${IMAGE_REPO_NAME}:${IMAGE_TAG}"""

                       // sh "docker push 416668258315.dkr.ecr.us-east-1.amazonaws.com/msp-repo:latest"
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
    
        
    
    