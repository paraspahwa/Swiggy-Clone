pipeline{
     agent any
     tools{
         jdk 'jdk17'
         nodejs 'node16'
     }
     environment {
         SCANNER_HOME=tool 'sonar-scanner'
     }
     stages {
         stage('clean workspace'){
             steps{
                 cleanWs()
             }
         }
         stage('Checkout from Git'){
             steps{
                 git branch: 'main', url: 'https://github.com/paraspahwa/Swiggy-Clone.git'
             }
         }
         stage("Sonarqube Analysis "){
             steps{
                 withSonarQubeEnv('sonar-server') {
                     sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Swiggy \
                     -Dsonar.projectKey=Swiggy '''
                 }
             }
         }
         stage("quality gate"){
            steps {
                 script {
                     waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token' 
                 }
             } 
         }
         stage('Install Dependencies') {
             steps {
                 sh "npm install"
             }
         }
         stage('OWASP FS SCAN') {
             steps {
                 dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                 dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
             }
         }
         stage('TRIVY FS SCAN') {
             steps {
                 sh "trivy fs . > trivyfs.txt"
             }
         }
         stage("Docker Build & Push"){
             steps{
                 script{
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){   
                        sh "docker build -t swiggy ."
                        sh "docker tag swiggy paraspahwa/swiggy:latest "
                        sh "docker push paraspahwa/swiggy:latest "
                     }
                 }
             }
         }
         stage("TRIVY"){
             steps{
                 sh "trivy image paraspahwa/swiggy:latest > trivyimage.txt" 
             }
         }
          stage('Deploy to kubernets'){
             steps{
                 script{
                     dir('Kubernetes') {
                         withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                                 sh 'kubectl apply -f deployment.yml'
                                 sh 'kubectl apply -f service.yml'
                         }   
                     }
                 }
             }
         }
     }
     post {
     always {
        emailext attachLog: true,
            subject: "'${currentBuild.result}'",
            body: "Project: ${env.JOB_NAME}<br/>" +
                "Build Number: ${env.BUILD_NUMBER}<br/>" +
                "URL: ${env.BUILD_URL}<br/>",
            to: 'noobstergaming5289@gmail.com',
            attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
        }
    }
 }
