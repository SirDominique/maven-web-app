pipeline {
    agent any

    tools {
        maven "maven3.9.6"
    }

    stages {
        stage('Git clone') {
            steps {
                git 'https://github.com/SirDominique/maven-web-app.git'
            }
        }

        stage ('Build and Package with maven') {
            steps {
                sh '''
                mvn clean
                mvn test
                mvn package
                '''
            }
        }

        stage ('Sonarqube Analysis') {
            environment {
                scannerHome = tool 'sonar5.0'
            }
            steps{
                script {
                    withSonarQubeEnv('sonarqube') {
                        sh "${ScannerHome}/bin/sonar-scanner -Dsonar.projectKey=jomacs"
                    }
                }
            }
        }

        stage ('Upload to Nexus') {
            steps {
                nexusArtifactUploader artifacts: [[artifactId: '01-maven-web-app', classifier: '', file: '/var/lib/jenkins/workspace/maven-web-app/target/maven-web-app.war', type: 'war']], credentialsId: 'nexus-id', groupId: 'in.ashokit', nexusUrl: '35.94.144.94:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'webapp-release', version: '3.0-RELEASE'
            }
        }

        stage ('Deploy to Tomcat') {
            steps {
                deploy adapters: [tomcat9(credentialsId: 'tomcat-credentials', path: '', url: 'http://34.219.120.95:8080/')], contextPath: null, war: 'target/*.war'
            }
        }
    }
}