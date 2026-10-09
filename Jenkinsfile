pipeline {
    agent any

    environment {
        SCANNER_HOME = tool 'sonar-Scanner'
        NVD_API_KEY = credentials('nvd-api-key')  // Jenkins secret text credential
    }

    tools {
        maven 'maven3'
        jdk 'JDK-21'
    }

    stages {
        stage('git checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/ShivNaik26/Ekart.git'
            }
        }

        stage('compile') {
            steps {
                sh "mvn compile"
            }
        }

        stage('unit tests') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }

        stage('SonarQube analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh "${env.SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=EKART \
                        -Dsonar.projectName=EKART \
                        -Dsonar.java.binaries=target/classes"
                }
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                  withCredentials([string(credentialsId: 'nvd-api-key', variable: 'NVD_API_KEY')]) {
                    dependencyCheck additionalArguments: "--nvdApiKey=$NVD_API_KEY",
                                    odcInstallation: 'dependency-check'
             }
        }
        }

        stage('Build') {
            steps {
                sh "mvn package -DskipTests=true"
            }
        }

        stage('deploy to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'd93e12f0-e46b-4c14-8ae0-b598660ac03d', jdk: 'JDK-21', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy -DskipTests=true"
                }
            }
        }
        

        stage('build and Tag docker image') {
            steps {
                script {
                        sh "docker build -t shiv173/ekart:latest -f docker/Dockerfile ."
                    }
            }
        }

        stage('Push image to Hub'){
    steps{
        script{
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

            sh 'docker push shiv173/ekart:latest'
        }
    }
}
        stage('EKS and Kubectl configuration'){
            steps{
                script{
                    sh 'aws eks update-kubeconfig --region ap-south-1 --name aws-devops-eks'
                }
            }
        }
        stage('Deploy to k8s'){
            steps{
                script{
                    sh 'kubectl apply -f deploymentservice.yml'
                }
            }
        }
    }

}
