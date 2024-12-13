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
    }

    stages {
        
        stage('Clone Repository') {
            steps {
                git branch: "${env.GIT_BRANCH}",
                    url: "${env.GIT_REPO}",
                    // credentialsId: 'github-token'  Replace with your Jenkins GitHub credentials ID
            }
        }

        stage('Build') {
            steps {
                sh "${env.MAVEN_HOME}/bin/mvn clean install"
            }
        }

         stage('Run Sonarqube') {
            environment {
                scannerHome = tool 'sonar-scanner';
            }
            steps {
              withSonarQubeEnv(credentialsId: 'sonar-token', installationName: 'sonar-server') {
                sh "${scannerHome}/bin/sonar-scanner -Dproject.settings=sonar-project.properties"
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
                        mvn deploy -s $SETTINGS_FILE
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
