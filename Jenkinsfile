pipeline {
    agent any

    tools {
        maven 'Maven'
    }

    options {
        skipDefaultCheckout(true)
    }

    environment {
        SONARQUBE_URL = 'http://3.109.144.111:30900/'
        SONARQUBE_TOKEN = credentials('Sonar-token-id')
        NEXUS_REPO_URL = 'http://15.206.79.47:32000/repository/maven-releases/'
        NEXUS_DOCKER_REPO = "<NEXUS_HOST>:3200"
        MAVEN_CREDENTIALS_ID = 'maven-settings'
        NEXUS_HOST = '15.206.79.47'
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
                        mvn deploy -DaltDeploymentRepository=nexus::default::${NEXUS_REPO_URL} --settings $SETTINGS_XML
                    """
                }
            }
        }

        stage('Download Artifact from Nexus') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'nexus-credentials', usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
                    sh """
                        wget --user=$NEXUS_USER --password=$NEXUS_PASS \
                        ${NEXUS_REPO_URL}com/example/simple-java-app/${VERSION}/simple-java-app-${VERSION}.jar \
                        -O app.jar
                    """
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ${NEXUS_DOCKER_REPO}/simple-java-app:${VERSION} .
                  """
            }
        }

        stage('Push Docker Image to Nexus') {
            steps {
                   sh "docker push ${NEXUS_DOCKER_REPO}/simple-java-app:${VERSION}"
                
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
