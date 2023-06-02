pipeline{
    agent any
    environment{
        VERSION = "${env.BUILD_ID}"
    }
    stages{
        stage('Sonar Quality Check'){
            agent{
                docker {
                    image 'maven'
                    args '-u root'
                }
            }
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'cicd3') {
                      sh 'mvn clean package sonar:sonar'
                    }
                }
            }
        }
        stage('Quality Gate Status'){
            steps{
                script{
                    waitForQualityGate abortPipeline: false, credentialsId: 'cicd3'
                }
            }
        }
        stage('docker Build and Push to Nexus Repo'){
           steps{
               script{
                   withCredentials([string(credentialsId: 'nexus-pass', variable: 'nexus_pass')]) {
                   sh '''
                    docker build -t 3.90.0.1:8083/springapp:${VERSION}

                    docker login -u admin -p $nexus_pass 3.90.0.1:8083

                    docker push 3.90.0.1:8083/springapp:${VERSION}

                    docker rmi 3.90.0.1:8083/springapp:${VERSION}
                    '''
                   }
               }
           }
        }
    }
}