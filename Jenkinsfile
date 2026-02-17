pipeline {
    agent any
    
    tools {
        maven "Maven"
    }
    stages {
        stage('Clean workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Git clone') {
            steps {
                git branch: 'create-dev', url: 'https://github.com/akhileshkulhade/java-maven-app-v1.git'
            }
        }
        stage('maven war file build') {
            steps {
               sh 'mvn clean package'
            }
        }
         stage('Docker images/conatiner remove') {
            steps {
                script{
                        sh '''
                        docker stop javamavenappdev_container || true
                        docker rm javamavenappsdev_container || true
                        docker rmi javamavenappdev || true
                        docker rmi akhik/javamavenappdev:latest || true
                        '''

                }  
            }
        }        
        stage('Docker images - Push to dockerhub') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker_creds'){
                
                        sh '''docker build -t javamavenappdev .
                        docker tag javamavenappdev akhik/javamavenappdev:latest
                        docker push  akhik/javamavenappdev:latest'''
                      } 
                }
            }
        }
        stage('docker container of app') {
            steps {
               sh 'docker run -d -p 9001:8080 --name javamavenappdev_container -t akhik/javamavenappdev:latest'
            }
        }
        
    }
    post {
    always {
        script {
            def buildStatus = currentBuild.currentResult
            def buildUser = currentBuild.getBuildCauses('hudson.model.Cause$UserIdCause')[0]?.userId ?: 'Github User'
            
            emailext (
                subject: "Pipeline ${buildStatus}: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                    <p>This is a Jenkins maven CICD pipeline status.</p>
                    <p>Project: ${env.JOB_NAME}</p>
                    <p>Build Number: ${env.BUILD_NUMBER}</p>
                    <p>Build Status: ${buildStatus}</p>
                    <p>Started by: ${buildUser}</p>
                    <p>Build URL: <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                """,
                to: 'mohdaseemakram19@gmail.com',
                from: 'mohdaseemakram19@gmail.com',
                replyTo: 'mohdaseemakram19@gmail.com',
                mimeType: 'text/html',
                attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
            )
           }
       }

    }
}
