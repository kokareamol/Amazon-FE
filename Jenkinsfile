pipeline {
    agent any
    
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }

    environment {
        SCANNER_HOME=tool 'sonar-server'
    }
    
    
    stages {
        stage('git checkout') {
            steps {
             git branch: 'main', url: 'https://github.com/kokareamol/Amazon-FE.git'   
            }
        }
        
        stage('sonarqube qulity check') {
            steps {
             withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=amazon \
                    -Dsonar.projectKey=amazon '''
                    
                  }
                }   
            }
            
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token' 
                }
            } 
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
            
        
        
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
            
            
    
        stage('docker build and push') {
            steps {
             script {
                 withDockerRegistry(credentialsId: 'docker', toolName: 'docker-server') {
                  sh "docker build -t amazon ."
                  sh "docker tag amazon kokareamol/amazon:latest"
                  sh "docker push kokareamol/amazon:latest"
            }
        } 
           
        
    }
}
        stage("Trivy image scan"){
          steps{
                 sh "trivy image kokareamol/amazon:latest > trivyimage.txt"
            }
        }
        
        
        stage('Deploy to container'){
            steps{
                sh 'docker run -d --name amazon -p 3000:3000 kokareamol/amazon:latest'
            }
        }


}
}
        
