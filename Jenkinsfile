pipeline {
    agent any

tools{
      maven  'maven3'
      jdk 'jdk17'
}


environment{
    SCANNER_HOME= tool 'sonar-scanner'
}

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/darien9797/secret-santa.git'
            }
        }
        
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        
         stage('OWASP filesystem scanner') {
            steps {
                dependencyCheck additionalArguments: '', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
         stage('Sonarqube') {
            steps {
                withSonarQubeEnv('sonar') {
                sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=secret-santa -Dsonar.projectKey=secret-santa \
                -Dsonar.java.binaries=target/classes '''
                }
            }
        }
        
        stage('Build Application & Push Artifact') {
            steps {
               withMaven(globalMavenSettingsConfig: '', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: 'maven-settings-default', traceability: true) {
                     sh "mvn deploy"
               }
            }
        }
        
         stage('Docker Build & Tag') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker build -t darien97/secret-santa:latest ."
                   }
                }
            }
        }
        
       stage('Trivy Image Scan') {
            steps {
                sh "trivy image --format table -o trivy-report.html darien97/secret-santa:latest "
            }
        }
        
         stage('Docker Push') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker push darien97/secret-santa:latest"
                   }
                }
            }
        }
        
        stage('K8-Deploy') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8-auth', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.36.202:6443') {
                    sh "kubectl apply -f k8-dep-svc.yml"
                    sh "kubectl get svc -n webapps"
               }
            }
        }
        
        
        
    }
}
