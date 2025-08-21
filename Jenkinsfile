pipeline {
    agent any

    tools {
        maven 'maven'              // Jenkins -> Global Tool Config -> Maven name
        jdk 'java8'                // Jenkins -> Global Tool Config -> JDK name
    }

    environment {
        SONARQUBE = credentials('sonarqube-token')  // SonarQube token
        NEXUS = credentials('nexus-credentials')    // Nexus credentials
        TOMCAT = credentials('tomcat')             // Tomcat credentials
        SONAR_SCANNER_HOME = tool 'sonarscanner'   // Jenkins tool name for SonarScanner
    }

    stages {
        stage('Git Clone') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/alsamdevops/srping3.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') { // Jenkins System Config -> SonarQube server name
                    sh """
                        ${SONAR_SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=spring3 \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=http://192.168.137.9:9000 \
                        -Dsonar.login=$SONARQUBE
                    """
                }
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package -DskipTests=true'
            }
        }

        stage('Upload to Nexus') {
            steps {
                sh """
                    mvn deploy:deploy-file \
                      -DgroupId=com.spring3.app \
                      -DartifactId=spring3-app \
                      -Dversion=1.0.0 \
                      -Dpackaging=war \
                      -Dfile=target/*.war \
                      -DrepositoryId=nexus \
                      -Durl=http://192.168.137.9:8081/repository/spring/ \
                      -DgeneratePom=true \
                      -DauthToken=$NEXUS
                """
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                sh """
                    curl -u $TOMCAT \
                      --upload-file target/*.war \
                      "http://192.168.137.10:8080/manager/text/deploy?path=/spring3&update=true"
                """
            }
        }
    }
}

