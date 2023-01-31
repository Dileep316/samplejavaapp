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
                        docker build -t 34.134.252.135:8083/springapp:${VERSION} .
                        docker login -u admin -p $docker_password 34.134.252.135:8083
                        docker push  34.134.252.135:8083/springapp:${VERSION}
                        docker rmi 34.134.252.135:8083/springapp:${VERSION}
                    '''
                }
            }
        }
    }
    stage('identifying misconfigs using datree in helm charts'){
            steps{
                script{
                    dir('kubernetes/') {
                        withEnv(['DATREE_TOKEN=ba2cdd8b-8659-4874-b045-ee5c80edf39e']) {
                            sh 'helm datree test myapp/'
                        }
                        
                    }
                }
            }
    }               
   stage("pushing the helm charts to nexus"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_password')]) {
                          dir('kubernetes/') {
                             sh '''
                                 helmversion=$( helm show chart myapp | grep version | cut -d: -f 2 | tr -d ' ')
                                 tar -czvf  myapp-${helmversion}.tgz myapp/
                                 curl -u admin:$docker_password http://34.134.252.135:8081/repository/helm-hosted/ --upload-file myapp-${helmversion}.tgz -v
                            '''
                          }
                    }
                }
            }
        }

    stage('manual approval'){
        steps{
            script{
                timeout(1) {
                        mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> Go to build url and approve the deployment request <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "dtestmail71@gmail.com";  
                        input( message: "Deploy ${params.project_name}?", ok: 'Deploy')
                    }
                }
           }
        }
        stage('Deploying application on k8s cluster') {
            steps {
                 dir('kubernetes/') {
                    sh 'helm upgrade --install --set image.repository="34.134.252.135:8083/springapp " --set image.tag="${VERSION} " myjavaapp myapp/ '
                 }
                
            }
            
        }
    }
    post {
		always {
			mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "dtestmail71@gmail.com";  
		 }
	   }
}