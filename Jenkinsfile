 def COLOR_MAP = [
    'SUCCESS': 'good', 
    'FAILURE': 'danger',
]
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
                        sh "${ScannerHome}/bin/sonar-scanner -Dsonar.projectKey=web-app -Dsonar.projectKey=maven-web-app"
                    }
                }
            }
        }
        
        stage ( 'Quality Gate') {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        
        stage ('Upload to Nexus') {
            steps {
                nexusArtifactUploader artifacts: [[artifactId: '01-maven-web-app', classifier: '', file: '/var/lib/jenkins/workspace/maven-web-app/target/maven-web-app.war', type: 'war']], credentialsId: 'nexusid', groupId: 'in.ashokit', nexusUrl: '35.86.94.236:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'mavenwebapp-release', version: '3.0-RELEASE'
            }
        }
        
        stage ('Deploy to Tomcat') {
            steps {
                deploy adapters: [tomcat9(credentialsId: 'tomcatid', path: '', url: 'http://52.34.74.158:8080/')], contextPath: null, war: 'target/*.war'
            }
        }

    }
    
    post {
        success {
            slackSend channel: 'good', color: 'good', message: "Build successful: ${currentBuild.fullDisplayName}"
        }
        failure {
            slackSend channel: 'good', color: 'danger', message: "Build failed: ${currentBuild.fullDisplayName}"
        }
    }
}    