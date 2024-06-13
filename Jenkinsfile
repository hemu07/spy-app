pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    environment {
        SCANNER_HOME = tool 'sonarqube'
    }

    stages {
        stage('git checkout') {
            steps {
                sh "git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/hemu07/spy-app.git'"
            }
        }

        stage('compile') {
            steps {
                sh 'mvn compile'
            }
        }

        stage('test') {
            steps {
                sh 'mvn test -DskipTests=true'
            }
        }

        stage('Trivy scan') {
            steps {
                sh 'trivy fs --format table -o trivy-fs-report.html .'
            }
        }

        stage('sonaqube analysis') {
            steps {
                withSonarQubeEnv(credentialsId: 'sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=spyApp -Dsonar.projectName=spyApp -Dsonar.java.binaries=. '''
                }
            }
        }

        stage('test') {
            steps {
                sh 'mvn package -DskipTests=true'
            }
        }

        stage('build image') {
            steps {
               /* sh 'trivy fs --format table -o trivy-fs-report.html .'*/
            }
        }
    }
}