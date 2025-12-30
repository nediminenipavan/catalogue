pipeline {
    agent {
        node {
            label 'AGENT-1'
        }
    }

    environment {
        COURSE    = "Jenkins"
        appVersion = ""
        ACC_ID    = "319625659730"
        PROJECT   = "roboshop"
        COMPONENT = "catalogue"
        GITHUB_OWNER = "nediminenipavan"
        GITHUB_REPO  = "catalogue"
        GITHUB_API   = "https://api.github.com"
    }

    options {
        timeout(time: 10, unit: 'MINUTES')
        disableConcurrentBuilds()
    }

    stages {

        stage('Read Version') {
            steps {
                script {
                    def packageJSON = readJSON file: 'package.json'
                    appVersion = packageJSON.version
                    echo "App Version: ${appVersion}"
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Unit Test') {
            steps {
                sh 'npm test'
            }
        }

        stage('Sonar Scan') {
            environment {
                scannerHome = tool 'sonar-8.0'
            }
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Dependabot Security Gate') {
            environment {
                GITHUB_TOKEN = credentials('GITHUB_TOKEN')
            }
            steps {
                sh '''
                response=$(curl -s \
                  -H "Authorization: token $GITHUB_TOKEN" \
                  -H "Accept: application/vnd.github+json" \
                  "$GITHUB_API/repos/$GITHUB_OWNER/$GITHUB_REPO/dependabot/alerts?per_page=100")

                high_critical_open_count=$(echo "$response" | jq '[.[] |
                  select(.state=="open" and
                  (.security_advisory.severity=="high" or .security_advisory.severity=="critical"))
                ] | length')

                echo "Open HIGH/CRITICAL alerts: $high_critical_open_count"

                if [ "$high_critical_open_count" -gt 0 ]; then
                  echo "❌ Blocking pipeline"
                  exit 1
                fi
                '''
            }
        }

        stage('Build Image') {
            steps {
                withAWS(region:'us-east-1', credentials:'aws-creds') {
                    sh """
                    aws ecr get-login-password --region us-east-1 |
                    docker login --username AWS --password-stdin ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com

                    docker build -t ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion} .
                    docker push ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion}
                    """
                }
            }
        }

        stage('Trivy Scan') {
            steps {
                sh """
                trivy image \
                --severity HIGH,CRITICAL,MEDIUM \
                --exit-code 1 \
                ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion}
                """
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success {
            echo 'Pipeline succeeded'
        }
        failure {
            echo 'Pipeline failed'
        }
        aborted {
            echo 'Pipeline aborted'
        }
    }
} 
