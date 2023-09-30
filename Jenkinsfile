pipeline {
    agent any
    
    tools{
        jdk 'jdk17'
        maven 'maven3'
    }
    environment{
        
        SCANNER_HOME= tool 'sonarqube'
    }
    stages {
        stage('git Checkout') {
            steps {
               git branch: 'main', url: 'https://github.com/sureshpolisetty/Ekart.git' 
            }
        }
         stage('Compile') {
            steps {
               sh "mvn compile"
            }
        }
         stage('Tests') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }
        stage('OWASP Scan') {
            steps {
               dependencyCheck additionalArguments: ' --scan ./ ', odcInstallation: 'DC'
                 dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Trivy') {
            steps {
                sh "trivy fs ."
            }
        }
             stage('Sonarqube') {
              steps {
                withSonarQubeEnv('sonarqube'){
                   sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Ekart \
                   -Dsonar.java.binaries=. \
                   -Dsonar.projectKey=Ekart '''
               }
            }
        }
         stage('Build') {
            steps {
                sh "mvn clean package -DskipTests=true"
            }
        }
        
        stage('Docker image build') {
            steps {
                sh'docker build -t shopping-cart .'
            }
        }
        
        stage("Push to DockerHub"){
            steps{
                withCredentials([usernamePassword(credentialsId:"docker",passwordVariable:"dockerHubPass",usernameVariable:"dockerHubUser")]){
                    sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPass}"
                    sh "docker tag shopping-cart ${env.dockerHubUser}/shopping-cart:latest"
                    sh "docker push ${env.dockerHubUser}/shopping-cart:latest"
                }
            }
        }
        
        stage('Docker Deploy'){
           steps{
               script{
                   withDockerRegistry(credentialsId:'docker',toolName:'docker'){
                       sh"docker run -d --name shopping-cart-cont -p 8070:8070 sureshpolisetty/shopping-cart:latest"
                   }
               }
           }
       }
       
      
    }
}
