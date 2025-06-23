pipeline {
    agent any

    tools {
        maven 'Maven'  // Make sure this matches what you've set up in Jenkins > Global Tool Configuration
    }

    options {
        skipDefaultCheckout(true) // Prevents Jenkins from doing its default Git checkout
    }

    environment {
        SONARQUBE_URL = 'http://sonarqube:9000'
        SONARQUBE_TOKEN = credentials('Sonar-token-id')  // Replace with actual ID from Jenkins credentials
        NEXUS_REPO_URL = 'http://13.235.51.64:32000/' // Cleaned URL — remove "#browse"
        MAVEN_CREDENTIALS_ID = 'maven-settings' // Jenkins credentials ID
    }

    stages {
        stage('Checkout Code') {
            steps {
                git credentialsId: 'Git hub', branch: 'master', url: 'https://github.com/Shreyasbhandesh/Java-app.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQubeServer') { // Name must match Jenkins > SonarQube server config
                    sh """
                        mvn sonar:sonar \
                            -Dsonar.projectKey=Java-app \
                            -Dsonar.host.url=$SONARQUBE_URL \
                            -Dsonar.login=$SONARQUBE_TOKEN
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Project') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('Tag Build') {
            steps {
                script {
                    def tag = "v1.0.${env.BUILD_NUMBER}"
                    sh """
                        git config user.email "jenkins@example.com"
                        git config user.name "Jenkins"
                        git tag $tag
                        git push origin $tag
                    """
                }
            }
        }

        stage('Deploy to Nexus') {
            steps {
                withCredentials([file(credentialsId: 'maven-settings', variable: 'SETTINGS_XML')]) {
                    sh """
                        mvn deploy -DaltDeploymentRepository=nexus::default::$NEXUS_REPO_URL \
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

