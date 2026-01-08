pipeline {
    agent any

    environment {
        PROJECT_KEY = "demo-project"
        VERSION     = "1.0-SNAPSHOT.${BUILD_NUMBER}"
        IMAGE_NAME  = "demo-project"
    }

    tools {
        maven 'MAVEN'
    }

    stages {

        stage('Git Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Amith373/demo-use-repo.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('Sonar Analysis') {
            steps {
                withSonarQubeEnv('Sonarqube') {
                    sh """
                        mvn clean verify sonar:sonar \
                        -Dsonar.projectKey=${PROJECT_KEY} \
                        -Dsonar.projectName=${PROJECT_KEY}
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
            post {
                success {
                    echo "Quality Gate Passed"
                }
                failure {
                    echo "Quality Gate Failed"
                }
                aborted {
                    echo "Quality Gate Aborted"
                }
            }
        }

        stage('Build with Version') {
            steps {
                sh """
                    mvn clean install \
                    -Drevision=${VERSION}
                """
            }
        }

        stage('Upload to Nexus') {
            steps {
                nexusArtifactUploader(
                    artifacts: [[
                        artifactId: 'my-app',
                        classifier: '',
                        file: 'target/my-app-1.0-SNAPSHOT.jar',
                        type: 'jar'
                    ]],
                    credentialsId: 'nexus-id-passwd',
                    groupId: 'com.mycompany.app',
                    nexusUrl: '13.233.201.49:8081',
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    repository: 'my-app',
                    version: '1.0-SNAPSHOT'
                )
            }
        }
        stage('docker image build'){
            agent {label 'node3'}
            steps{
                withCredentials([usernamePassword(credentialsId: 'nexus-cred',usernameVariable: 'NEXUS_USER',passwordVariable: 'NEXUS_PASS')])
                {  
                    sh """
                         docker build \
                         --build-arg NEXUS_URL=http://13.233.201.49:8081 \
                         --build-arg NEXUS_USER=$NEXUS_USER \
                         --build-arg NEXUS_PASS=$NEXUS_PASS \
                         --build-arg VERSION=${VERSION} \
                         -t ${IMAGE_NAME}:${VERSION} .
                     """
                }
           }
        }
    }
}
