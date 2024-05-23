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
                    sh '''
                    # Ensure the directory exists
                    mkdir -p ~/.docker/cli-plugins

                    # Fetch the latest release URL for docker-buildx-linux-amd64
                    BUILDX_URL=$(curl -s https://api.github.com/repos/docker/buildx/releases/latest | grep browser_download_url | grep linux-amd64 | cut -d '"' -f 4)
                    
                    # Download and setup Buildx
                    wget $BUILDX_URL -O ~/.docker/cli-plugins/docker-buildx
                    chmod +x ~/.docker/cli-plugins/docker-buildx

                    # Create and use a new builder instance
                    docker buildx create --name mybuilder --use
                    docker buildx inspect mybuilder --bootstrap
                    '''
                }
            }
        }
        stage('Building image and Tagging') {
            steps {
                script {
                    // Build and tag the Docker image using Buildx
                    sh """
                    docker buildx build --platform linux/amd64 -t msp-repo:latest --push .
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
                        sh "docker push 416668258315.dkr.ecr.us-east-1.amazonaws.com/msp-repo:latest"
                            
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
    
        
    
    