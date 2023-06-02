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
                    docker build -t 3.90.0.1:8083/springapp:${VERSION} .

                    docker login -u admin -p $nexus_pass 3.90.0.1:8083

                    docker push 3.90.0.1:8083/springapp:${VERSION}

                    docker rmi 3.90.0.1:8083/springapp:${VERSION}
                    '''
                   }
               }
           }
        }
        stage('Identifying Misconfigs using datree in helm charts'){
            steps{
                script{
                    dir('kubernetes/myapp') {
                        withEnv(['DATREE_TOKEN= e7584138a0-0556-7w00-y352486154654a1'])
                        sh 'helm datree test .'

                    }
                }
            }
        }
        stage('Pushing the helm chart to repo'){
            steps{
                script{
                     withCredentials([string(credentialsId: 'nexus-pass', variable: 'nexus_pass')]){
                        dir('kubernetes/myapp') {
                            sh '''
                            helmversion=$(helm show chart myapp | grep version | cut -d: -f 2 | tr -d ' ')
                            tar -czvf myapp-${helmversion}.tgz myapp/
                            curl -u admin:$nexus_pass http://3.90.0.1:8081/repository/helm-repo/ --upload-file myapp-${helmversion}.tgz -v
                            '''
                        }
                     }
                }
            }
        }
    }
}