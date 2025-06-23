pipeline {
    agent any

    environment {
        SONARQUBE_URL = 'http://sonarqube:9000'
        SONARQUBE_TOKEN = credentials('sonar-token')       // Jenkins secret
        NEXUS_REPO_URL = 'http://nexus:8081/repository/maven-releases/'
        MAVEN_CREDENTIALS_ID = 'nexus-credentials'         // Jenkins secret
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'master', url: 'https://github.com/Shreyasbhandesh/Java-app.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQubeServer') { // Jenkins > Manage > SonarQube servers
                    sh """
                    mvn sonar:sonar \
                        -Dsonar.projectKey=your-project \
                        -Dsonar.host.url=$SONARQUBE_URL \
                        -Dsonar.login=$SONARQUBE_TOKEN
                    """
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
                        // git config user.email "jenkins@example.com"
                        // git config user.name "Jenkins"
                        git tag $tag
                        git push origin $tag
                    """
                }
            }
        }

        stage('Deploy to Nexus') {
            steps {
                sh """
                mvn deploy -DaltDeploymentRepository=nexus::default::$NEXUS_REPO_URL \
                           --settings settings.xml
                """
            }
        }
    }

    post {
        failure {
            echo 'Pipeline failed.'
        }
        success {
            echo 'Build, scan, tag, and deploy successful!'
        }
    }
}
