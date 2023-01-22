pipeline{
    agent any 
    environment{
        VERSION = "${env.BUILD_ID}"
    }
    stages{
        stage("sonar quality check"){
        steps{
            script{
                withSonarQubeEnv(credentialsId: 'sonartoken') {
                    sh 'chmod +x gradlew'
                    sh './gradlew --warning-mode=all sonarqube'
                }

                timeout(time: 1, unit: 'HOURS') {
                      def qg = waitForQualityGate()
                      if (qg.status != 'OK') {
                           error "Pipeline aborted due to quality gate failure: ${qg.status}"
                      }
                    }
            }
        }  
    }
    stage("docker build and docker push"){
        steps{
            script{
                withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_password')]) {
                     sh '''
                        docker build -t 34.125.214.226:8083/springapp:${VERSION} .
                        docker login --username admin --password-stdin < ~/$docker_password 34.125.214.226:8083 
                        docker push  34.125.214.226:8083/springapp:${VERSION}
                        docker rmi 34.125.214.226:8083/springapp:${VERSION}
                    '''
                }
            }
        }
    }
}
}
