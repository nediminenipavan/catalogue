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
        stage('Build Image') {
            steps {
                 script{
                    sh """
                       docker build -t catalogue:${appversion} .
                       docker images 
                    """
                }   
            }
        }
        stage('Deploy') {
            //   input {
            //     message "Should we continue?"
            //     ok "Yes, we should."
            //     submitter "alice,bob"
            //     parameters {
            //         string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')
            //     }
            // }
            when {
                expression { "$params.DEPLOY" == "true" }
            }
            steps {
                script{
                    sh """
                       echo "Building" 
                    """
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