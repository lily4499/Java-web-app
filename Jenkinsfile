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

        stage('Run SonarQube') {
            environment {
                scannerHome = tool name: 'sonar-scanner'  // Adjust 'sonar-scanner' to your Jenkins scanner installation name
            }
            steps {
                withSonarQubeEnv('sonar-token') {  // Replace 'sonar-token' with your SonarQube credentials ID
                    sh """
                        ${scannerHome}/bin/sonar-scanner \
                        -Dproject.settings=sonar-project.properties
                    """
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

