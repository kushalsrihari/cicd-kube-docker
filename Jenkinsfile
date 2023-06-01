pipeline{
    agent any
    stages{
        stage('Sonar Quality Check'){
            agent{
                docker{
                    image 'maven'
                }
            }
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'cicd') {
                        sh 'mvn clean package sonar:sonar'
                    }
                }
            }
        }
    }
}