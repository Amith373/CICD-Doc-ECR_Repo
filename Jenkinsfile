pipeline{
    agent any
    tools {
        maven 'MAVEN'
    }
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
      stage('Deploy'){
            steps{
            withCredentials([usernamePassword(credentialsId: 'git_hub_new',usernameVariable:'GIT_USER',passwordVariable:'GIT_PASS')]){
            sh """ rm -rf calculator_deployment
              git clone https://${GIT_USER}:${GIT_PASS}@github.com/Amith373/CICD-Doc-ECR_Repo.git
              cd calculator_deployment/k8s
             sed -i "s|image: .*calculator-java.*|image: 316444450716.dkr.ecr.us-east-1.amazonaws.com/calculator-java:${VERSION}|Ig" sample.yaml
             git add sample.yaml
             git commit -m "updated the image" 
             git push origin main
            """
            }
        }
    }
 }
}
