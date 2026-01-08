pipeline{
    agent any
    environment{
        PROJECT_KEY="calculator-project"
        VERSION="1.0.${env.BUILD_NUMBER}"
        IMAGE_NAME="calculator-java"
    }
    stages{
        stage('SCM'){
            steps{
                git branch: 'main', url: 'https://github.com/Amith373/CICD-Doc-ECR_Repo.git'
            }
        }
        stage('sonar analysis'){
            steps{
                withSonarQubeEnv('sonar-k8s'){
                    sh """ mvn clean verify sonar:sonar \
                    -Dsonar.projectKey=${PROJECT_KEY} \
                    -Dsonar.projectName=${PROJECT_KEY} \
                    -Drevision=${VERSION}
                    """
                }
            }
        }
        stage('Quality gate validate'){
            steps{
                timeout(time: 5, unit: 'MINUTES'){
                    waitForQualityGate abortPipeline: true
                }
            }
                  post{
                    success{
                           echo "Quality Gate Passed"
                        }
                   failure{
                           echo "Quality Gate Failed"
                     }
                }
            }
        stage('Build'){
            steps{
                sh """ mvn clean install \
                       -Drevision=${VERSION} """
            }
        }
          stage('Nexus-artifactory'){
            steps{
                nexusArtifactUploader artifacts: [[artifactId: 'calculator-java', classifier: '', file: "target/calculator-java-${VERSION}.jar", type: 'jar']],
                    credentialsId: 'nexus-cred', groupId: 'com.example', nexusUrl: '52.54.112.218:32000', nexusVersion: 'nexus3', protocol: 
                    'http', repository: 'maven-releases', version: "${VERSION}"
            }
        }
    }
}
