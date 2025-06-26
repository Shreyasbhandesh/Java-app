pipeline {
    agent any

    tools {
        maven 'Maven' // Must match your Global Tool Configuration
    }

    options {
        skipDefaultCheckout(true)
    }

    environment {
        SONARQUBE_URL = 'http://3.109.144.111:30900/'
        SONARQUBE_TOKEN = credentials('Sonar-token-id')
        NEXUS_REPO_URL = 'http://15.206.79.47:32000/repository/maven-releases/'
        MAVEN_CREDENTIALS_ID = 'maven-settings'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git credentialsId: 'Git hub', branch: 'master', url: 'https://github.com/Shreyasbhandesh/Java-app.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQubeServer') {
                    sh """
                        mvn sonar:sonar \
                            -Dsonar.projectKey=Java-app \
                            -Dsonar.host.url=${SONARQUBE_URL} \
                            -Dsonar.login=${SONARQUBE_TOKEN}
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Project') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('Set Version') {
          steps {
            script {
               env.VERSION = "1.0.${env.BUILD_NUMBER}"
                  }

             sh """
              mvn versions:set -DnewVersion=${VERSION}
              mvn versions:commit
               """
               }
           }


        stage('Deploy to Nexus') {
            steps {
                withCredentials([file(credentialsId: 'maven-settings', variable: 'SETTINGS_XML')]) {
                    sh """
                        mvn deploy -DaltDeploymentRepository=nexus::default::${NEXUS_REPO_URL} \
                                   --settings $SETTINGS_XML
                    """
                }
            }
        }
    }

    post {
        success {
            echo '✅ Build, scan, tag, and deploy successful!'
        }
        failure {
            echo '❌ Pipeline failed.'
        }
    }
}

