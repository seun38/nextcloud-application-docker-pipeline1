pipeline {

    tools{

        maven 'maven3.9.1'
    }
    agent any

    environment {
        registry = "618348145639.dkr.ecr.us-east-1.amazonaws.com/liontech-demo-automation"
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/Lion-Technology-Solutions/jenkins-helm-docker-eks.git']])
            }
        }
        
        stage ("Build JAR") {
            steps {
                sh "mvn clean install"
            }
        }
        
        stage ("Build Image") {
            steps {
                script {
                    dockerImage = docker.build registry
                    dockerImage.tag("$BUILD_NUMBER")
                }
            }
        }
        
        stage ("Push to ECR") {
            steps {
                script {
                    sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 618348145639.dkr.ecr.us-east-1.amazonaws.com'
                    sh 'docker push 618348145639.dkr.ecr.us-east-1.amazonaws.com/liontech-demo-automation:$BUILD_NUMBER'
                    
                }
            }
        }
        stage('connnect'){
            steps{
                withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'prince', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                 sh ('kubectl apply -f  release/kubernetes-manifests.yaml')
}
            }
        }
        
        stage("helmdeploy"){
            steps{

                script{
                    sh "helm upgrade first --install mychart --namespace liontech-dev --set image.tag=$BUILD_NUMBER"
                }
            }
        }
        
        
        stage ("Helm package") {
            steps {
                    sh "helm package springboot"
                }
            }
                
        stage ("Helm install") {
            steps {
                    sh "helm upgrade myrelease-21 springboot-0.1.0.tgz"
                }
            }
    }
}