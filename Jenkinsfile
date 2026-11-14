pipeline {
    agent any

    environment {
        SCANNER_HOME = tool 'sonar-Scanner'
        NVD_API_KEY = credentials('nvd-api-key')
    }

    tools {
        maven 'maven3'
        jdk 'JDK-21'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/Shiv173/Ecart-Shopping-App.git'
            }
        }

        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }

        stage('Unit Tests') {
            steps {
                sh "mvn test"
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh "${env.SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=EKART \
                        -Dsonar.projectName=EKART \
                        -Dsonar.java.binaries=target/classes"
                }
            }
        }

        stage('SonarQube Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                withCredentials([string(credentialsId: 'nvd-api-key', variable: 'NVD_API_KEY')]) {
                    dependencyCheck additionalArguments: "--nvdApiKey=$NVD_API_KEY",
                                    odcInstallation: 'dependency-check'
                    dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                }
            }
        }

        stage('Build Artifact') {
            steps {
                sh "mvn package -DskipTests=true"
            }
        }

        stage('Deploy to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'd93e12f0-e46b-4c14-8ae0-b598660ac03d', jdk: 'JDK-21', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy -DskipTests=true"
                }
            }
        }

        stage('Build and Tag Docker Image') {
            steps {
                script {
                    sh "docker build -t shiv173/ekart:${env.BUILD_NUMBER} -t shiv173/ekart:latest -f docker/Dockerfile ."
                }
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'dockerhub-pwd-new',
                        usernameVariable: 'DOCKERHUB_USER',
                        passwordVariable: 'DOCKERHUB_PASS'
                    )]) {
                        sh '''
                            set +x
                            echo "$DOCKERHUB_PASS" | docker login --username "$DOCKERHUB_USER" --password-stdin
                        '''
                    }
                    sh "docker push shiv173/ekart:${env.BUILD_NUMBER}"
                    sh "docker push shiv173/ekart:latest"
                }
            }
        }

        stage('EKS and Kubectl Configuration') {
            steps {
                script {
                    sh 'aws eks update-kubeconfig --region ap-south-1 --name aws-devops-eks'
                }
            }
        }

        stage('Deploy to EKS with Zero-Downtime Rolling Update') {
            steps {
                script {
                    sh 'kubectl apply -f deploymentservice.yml'
                    sh 'kubectl rollout restart deployment/ekart-deployment'
                    sh 'kubectl rollout status deployment/ekart-deployment'
                }
            }
        }
    }
}
