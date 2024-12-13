pipeline {
    agent any

    environment {
        MAVEN_HOME = '/usr/share/maven'
        GIT_REPO = 'https://github.com/lily4499/Java-web-app.git'  // Corrected repository URL
        GIT_BRANCH = 'main'  // Replace with your branch name if different
        TOMCAT_URL = 'http://localhost:8081/manager/text'
        TOMCAT_USER = 'admin'
        TOMCAT_PASSWORD = 'abc123'
        CONTAINER_ID = 'tomcat' // Replace with your container ID or name
        SONARQUBE_URL = 'http://localhost:9000' // Update with your SonarQube URL
        SONARQUBE_TOKEN = credentials('sonar-token') // Jenkins credential ID for SonarQube token
    }

    stages {
        
        stage('Clone Repository') {
            steps {
                git branch: "${GIT_BRANCH}",
                    url: "${GIT_REPO}"
                    // Uncomment and replace with your Jenkins GitHub credentials ID if required:
                    // credentialsId: 'github-token'
            }
        }

        stage('Build') {
            steps {
                sh "${MAVEN_HOME}/bin/mvn clean install"
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQubeServer') { // Replace 'SonarQubeServer' with the name configured in Jenkins
                    sh """
                    mvn sonar:sonar \
                    -Dsonar.projectKey=my-app \
                    -Dsonar.host.url=${SONARQUBE_URL} \
                    -Dsonar.login=${SONARQUBE_TOKEN}
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Deploy App via Tomcat Manager') {
            steps {
                sh """
                    curl -u ${TOMCAT_USER}:${TOMCAT_PASSWORD} -T target/your-app.war \
                    ${TOMCAT_URL}/deploy?path=/your-app&update=true
                """
            }
        }
       
        stage('Deploy to Artifactory') {
            steps {
                withCredentials([file(credentialsId: 'jfrog-settings-secret', variable: 'SETTINGS_FILE')]) {
                    sh """
                        ${MAVEN_HOME}/bin/mvn deploy -s $SETTINGS_FILE
                    """
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed.'
        }
    }
}

