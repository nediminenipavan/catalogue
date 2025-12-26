pipeline {
    // These are pre-build sections 
     agent {
        node {
            label 'AGENT-1'
        }
    }
     environment {
         COURSE = "Jenkins"
         appVersion = ""
         ACC_ID = "319625659730"
         PROJECT = "roboshop"
         COMPONENT = "catalogue" 
    }
     options {
        timeout(time: 10, unit: 'MINUTES') 
        disableConcurrentBuilds()
    }
    // This is build section 
    stages {
        stage('Read Version') {
            steps {
                    script{
                       def packageJSON = readJSON file: 'package.json'
                       appVersion = packageJSON.version
                       echo "app version: ${appVersion}"
                }   
            }
        }
        stage('Install Dependencies') {
            steps {
                 script{
                    sh """
                       npm install 
                    """
                }   
            }
        }
        stage('Unit Test') {
            steps {
                 script{
                    sh """
                       npm test 
                    """
                }   
            }
        }
        //Here you need to select scanner tool and send the analysis to server 
        stage('Sonar Scan'){
            environment {
                def scannerHome = tool 'sonar-8.0'
            }
            steps {
                script{
                    withSonarQubeEnv('sonar-server') {
                        sh "${scannerHome}/bin/sonar-scanner"
                        }
                    }
                }
        }
        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                // Wait for the SonarQube quality gate status
                // abortPipeline: true will fail the Jenkins build if the QG fails
                waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Build Image') {
            steps {
                 script{
                    withAWS(region:'us-east-1',credentials:'aws-creds') {
                        sh """
                            aws ecr get-login-password --region us-east-1 \
                            | docker login --username AWS --password-stdin ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com
                            docker build -t ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion} .
                            docker images 
                            docker push ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion}
                        """
                    }
                 }
            }
        }
    }
    post{
        always{
            echo 'I will always say Hello again!'
            cleanWs()
        }
        success {
            echo 'I will run if success'
        }
        failure {
            echo 'I will run if failure'
        }
        aborted {
            echo 'pipeline is aborted'
        } 
    }
}       