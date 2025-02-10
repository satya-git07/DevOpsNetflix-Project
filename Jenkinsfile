pipeline {
    agent any

    environment {
        // Define values as environment variables for better maintainability
        CLUSTER_NAME = 'my-cluster1'
        ZONE = 'us-west3'
        PROJECT_ID = 'satyanarayana'
        GIT_BRANCH = 'main'
    }

    stages {
        stage('Git Checkout') {
            steps {
                // Checkout the repository using the defined branch
                git url: 'https://github.com/satya-git07/DevOpsNetflix-Project.git', branch: "${GIT_BRANCH}"
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker-credentials', toolName: 'docker'){   
                       sh "docker build -t netflix ."
                       sh "docker tag netflix:latest satyadockerhub07/netflix:tagname"
                       sh "docker push satyadockerhub07/netflix:${BUILD_NUMBER}"
                    }
                }
            }
        }
        
        stage('Terraform Init') {
            steps {
                // Initialize Terraform
                sh 'terraform init'
                sh 'ls -la'
            }
        }
        stage('Terraform Apply') {
            steps {
                // Authenticate and apply Terraform changes
                withCredentials([file(credentialsId: 'gcp-sa', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    sh 'terraform apply -auto-approve'
                }
            }
        }
       
        
        stage('Deploy to GKE') {
            steps {
                // Authenticate and deploy to GKE
                withCredentials([file(credentialsId: 'gcp-sa', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    script {
                        sh 'gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS'
                        sh "gcloud container clusters get-credentials ${CLUSTER_NAME} --zone ${ZONE} --project ${PROJECT_ID}"
                        sh 'kubectl apply -f ./Kubernetes/deployment.yml' 
                        sh 'kubectl apply -f ./Kubernetes/service.yml'
                    }
                }
            }
        }
    }
 
post {
        always {
            cleanWs()
        }
    }
}
