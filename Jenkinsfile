pipeline{
    agent any
    tools{
        maven 'maven3'
    }
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
    }
    stages {
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/Nishantbadhiye/Multi-Tier-with-Database.git'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs --format table -o  trivyfs.html . "
            }
        }
        stage('Build') {
            steps {
                sh "mvn clean package"
            }
        }        
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('Sonar') {
                    sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Multitier \
                    -Dsonar.projectKey=Multitier -Dsonar.java.binaries=target"
                }
            }
         }
        
        stage('Publish To Nexus ') {
            steps {
                withMaven(globalMavenSettingsConfig:'settings-maven', jdk:'', maven:'maven3'){ 
                    sh "mvn deploy"
                }
            }
        }

        stage('Docker Build & Push'){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker'){   
                       sh "docker build -t nishantbadhiye/bankapp:latest ."
                       sh "docker push nishantbadhiye/bankapp:latest"
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image --format table -o trivyimage.html nishantbadhiye/bankapp:latest" 
            }
        }
        stage('Deploy to Kubernetes'){
            steps{
                 withKubeConfig(caCertificate:'',clusterName:'devops-cluster',
                     sh "kubectl apply -f ds.yml -n webapps"
                     sleep 30
            }
        }
    
        stage('Verify Deployment'){
            steps{
                 withKubeConfig(caCertificate:'', clusterName:'devops-cluster', contextName:'',credentialsId:
                     sh "kubectl get pods -n webapps"
                     sh "kubectl get svc -n webapps"
            }
        }
    }

