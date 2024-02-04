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
                git branch: 'master', url: 'https://github.com/RAMESHKUMARVERMAGITHUB/flask_mongodb_dockerized_app.git'
            }
        }
        
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=flask_mongodb_dockerized_app \
                    -Dsonar.projectKey=flask_mongodb_dockerized_app'''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar'
                }
            }
        }
        // stage('Install Dependencies') {
        //     steps {
        //         sh "npm install"
        //     }
        // }
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
                       sh "docker build -t rameshkumarverma/flask_mongodb_dockerized_app:latest ."
                       // sh "docker tag flask_mongodb_dockerized_app rameshkumarverma/flask_mongodb_dockerized_app:latest "
                       sh "docker push rameshkumarverma/flask_mongodb_dockerized_app:latest"
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image rameshkumarverma/flask_mongodb_dockerized_app:latest > trivyimage.txt"
            }
        }
        // stage("deploy_docker"){
        //     steps{
        //         sh "docker run -d --name flask_mongodb_dockerized_app -p 0000:0000 rameshkumarverma/flask_mongodb_dockerized_app:latest"
        //     }
        // }
        stage("docker-deploy"){
            steps{
                sh "docker-compose up -d"
            }
        }
        // stage('Deploy to kubernets'){
        //     steps{
        //         script{
        //             // dir('K8S') {
        //                 withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
        //                         sh 'kubectl apply -f deployment-service.yml'
        //                         // sh 'kubectl apply -f service.yml'
        //                 }
        //             // }
        //         }
        //     }
        // }
    }
}
