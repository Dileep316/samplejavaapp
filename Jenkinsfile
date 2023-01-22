pipeline{
    agent any 
    stage("sonar quality check"){
        agent {
            docker {
                image 'openjdk:11'
            }
        }
        
        steps{
            script{
                withSonarQubeEnv(credentialsId: 'sonartoken') {
                    sh 'chmod +x gradlew'
                    sh './gradlew sonarqube'
                }
            }
        }
       
    }
}