pipeline{
    agent any
    tools {
        maven "maven3"
        jdk "jdk17"
    }
    environment{
        SCANNER_HOME = tool "sonar-scanner"
    }
      
      stages{
          
          stage("Git Checkout"){
              steps{
                  git branch: 'main', url: 'https://github.com/SomasekharSunkari/Ekart_Jenkin_piplinedemo.git'
              }
          }
           stage("Code Compilation"){
              steps{
                    sh "mvn compile"
              }
          }
          stage("Unit Tests"){
              steps{
                  sh "mvn test -DskipTests=true"
              }
          }
          stage("Sonarqube Analysis"){
              steps{
                  withSonarQubeEnv('sonar-server') {
                      sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=EKART -Dsonar.projectName=EKART \
                      -Dsonar.java.binaries=.
                      '''
                        
                }
              }
          }
          stage("OWASP Dependeny Check"){
              steps{
                  dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'dc'
              }
          }
          stage("Build"){
              steps{
                  sh "mvn package -DskipTests=true"
              }
          }
          stage("Deplot to Nexus"){
              steps{
                  withMaven(globalMavenSettingsConfig: 'maven-releases', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                            
                            sh "mvn deploy -DskipTests=true"
                        }
              }
          }
          stage("Docker build & tag"){
              steps{
              withDockerRegistry(credentialsId: 'Dockhub-cred',url:"https://index.docker.io/v1/") {
                        sh "docker build -t sunkarisomasekhar/ekart:latest -f docker/Dockerfile ."
              }
                        }
          }
          stage("Trivy Image Scan"){
              steps{
              sh "trivy image sunkarisomasekhar/ekart:latest > trivy-report.txt"
              }
          }
          stage("Push Image to DockerHub"){
              steps{
              sh "docker push sunkarisomasekhar/ekart:latest"
              }
          }
          stage("K8s Deployment"){
              steps{
                  withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: 'java-webapp', restrictKubeConfigAccess: false, serverUrl: 'https://127.0.0.1:38775') {
    // some block
    sh "kubectl apply -f deploymentservice.yml"
                  sh "kubectl get svc -n java-webapp"
}
                  
              }
              
          }
      }
}
