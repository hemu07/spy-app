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
                echo "git checkout in progress.."
            }
        }

        stage('compile') {
            steps {
                sh 'mvn compile'
            }
        }

        stage('test app') {
            steps {
                sh 'mvn test -DskipTests=true'
            }
        }

        stage('Trivy scan') {
            steps {
                sh 'trivy fs --format table -o trivy-fs-report.html .'
            }
        }

        stage('sonarqube analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=spyApp -Dsonar.projectName=spyApp -Dsonar.java.binaries=. '''
                }
            }
        }

        stage('build app') {
            steps {
                sh 'mvn package -DskipTests=true'
            }
        }

        stage('deploy to nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'maven', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh 'mvn package -DskipTests=true && mvn clean deploy -DskipTests=true'
                }
            }
        }

        stage('build image') {
            steps {
                echo "building image"
             
            }
        }
    }
}