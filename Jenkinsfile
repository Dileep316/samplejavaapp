pipeline{
    agent any 
    stages{
        stage("sonar quality check"){
        
        
        steps{
            script{
                withSonarQubeEnv(credentialsId: 'sonartoken') {
                    sh 'chmod +x gradlew'
                    sh './gradlew --warning-mode=all sonarqube'
                }
            }
        }
       
    }
  }
}
