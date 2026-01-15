pipeline{
    agent any
    tools {
        maven 'MAVEN'
    }
    environment{
        PROJECT_KEY="calculator-project"
        VERSION="v1.${env.BUILD_NUMBER}"
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
                    credentialsId: 'nexus-cred', groupId: 'com.example', nexusUrl: '98.81.161.115:32000', nexusVersion: 'nexus3', protocol: 
                    'http', repository: 'Calculator-app', version: "${VERSION}"
            }
        }
        stage('docker image build'){
            agent any
            steps{
                withCredentials([usernamePassword(credentialsId: 'nexus-cred',usernameVariable: 'NEXUS_USER',passwordVariable: 'NEXUS_PASS')])
                {  
                    sh """
                         docker build \
                         --build-arg NEXUS_URL=http://98.81.161.115:32000 \
                         --build-arg NEXUS_USER=$NEXUS_USER \
                         --build-arg NEXUS_PASS=$NEXUS_PASS \
                         --build-arg VERSION=${VERSION} \
                         -t ${IMAGE_NAME}:${VERSION} .
                     """
                }
           }
        }
        stage('Image to ECR'){
            steps{
                withAWS(credentials:'jenkins-ecr',region:'us-east-1')
                {
                    sh """
                        aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 316444450716.dkr.ecr.us-east-1.amazonaws.com
                        docker tag ${IMAGE_NAME}:${VERSION} 316444450716.dkr.ecr.us-east-1.amazonaws.com/calculator-java:${VERSION}
                        docker push 316444450716.dkr.ecr.us-east-1.amazonaws.com/calculator-java:${VERSION}
                        """
                }
            }
        }
      stage('Update Image in GitRepo') {
    steps {
        withCredentials([
            usernamePassword(
                credentialsId: 'github-creds',
                usernameVariable: 'GIT_USER',
                passwordVariable: 'GIT_TOKEN'
            )
        ]) {
            sh """
                rm -rf ArgoCD-Demo-Project
                git clone https://${GIT_USER}:${GIT_TOKEN}@github.com/Amith373/ArgoCD-Demo-Project.git
                cd ArgoCD-Demo-Project
                sed -i 's|image:.*|image: ${ECR_REPO}:${VERSION}|' calculator.yaml
                git add calculator.yaml
                git commit -m "Update image to ${VERSION}"
                git push
            """
           }
         }
       }
     }
}
