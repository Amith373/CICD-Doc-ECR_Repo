pipeline {
   agent {label 'node3'}

    environment {
        PROJECT_KEY = "my-app"
        VERSION     = "1.0-SNAPSHOT.${BUILD_NUMBER}"
        IMAGE_NAME  = "my-app"
    }

    tools {
        maven 'MAVEN'
    }

    stages {

        stage('Git Checkout') {
            steps {
               git branch: 'main', url: 'https://github.com/Amith373/CICD-Doc-ECR_Repo.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('Sonar Analysis') {
            steps {
                withSonarQubeEnv('Sonar') {
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
    }
}
