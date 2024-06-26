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

        stage('Trivy scan file system') {
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
                    sh 'mvn clean deploy -DskipTests=true'
                }
            }
        }

        stage('build & tag docker image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker build -t hemu07/mission:latest ."
                    }
                }
            }
        }

        stage('Trivy scan image') {
            steps {
                sh 'trivy image --format table -o trivy-image-report.html hemu07/mission:latest'
            }
        }

        stage('publish docker image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker push hemu07/mission:latest"
                    }
                }
            }
        }

        stage('deploy to k8s') {
            steps {
                script {
                    withKubeConfig(caCertificate: '', clusterName: 'spy-app', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://16998441D9054663C943442635C1B9A1.gr7.us-east-1.eks.amazonaws.com') {
                        sh "kubectl apply -f deployment.yaml -n webapps"
                        sleep 60
                    }
                }
            }
        }

        stage('verify the deployment to k8s') {
            steps {
                script {
                    withKubeConfig(caCertificate: '', clusterName: 'spy-app', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://16998441D9054663C943442635C1B9A1.gr7.us-east-1.eks.amazonaws.com') {
                        sh "kubectl get pods -n webapps"
                        sh "kubectl get svc -n webapps"
                    }
                }
            }
        }
    }
}