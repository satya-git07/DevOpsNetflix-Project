pipeline {
    agent any

    environment {
        CLUSTER_NAME = 'my-cluster1'
        ZONE = 'us-west3'
        PROJECT_ID = 'satyanarayana'
        GIT_BRANCH = 'main'
        //SONARQUBE_HOST = 'http://192.168.2.109:9000'  // Your SonarQube Server URL
        //SONARQUBE_PROJECT_KEY = 'netflix'  // Your SonarQube Project Key
        //SONARQUBE_TOKEN = 'squ_d8a6704a71d077aff483843ba032f5ca800e3d42'  // Your SonarQube Token
    }

    stages {
        stage('Git Checkout') {
            steps {
                // Checkout the repository using the defined branch
                git url: 'https://github.com/satya-git07/DevOpsNetflix-Project.git', branch: "${GIT_BRANCH}"
            }
        }
        
        // SonarQube Analysis Stage using sonar-scanner
        stage('SonarQube Analysis') {
            steps {
                script {
                    // Run SonarQube analysis using sonar-scanner
                    sh """
                        sonar-scanner \
                            -Dsonar.projectKey='netflix' \
                            -Dsonar.sources=. \
                            -Dsonar.host.url='http://192.168.2.109:9000' \
                            -Dsonar.login='squ_d8a6704a71d077aff483843ba032f5ca800e3d42'
                    """
                }
            }
        }
        
        stage("Docker Build & Push") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-credentials', toolName: 'docker') {   
                        sh "docker build -t netflix ."
                        sh "docker tag netflix:latest satyadockerhub07/netflix:tagname"
                        sh "docker push satyadockerhub07/netflix:tagname"
                    }
                }
            }
        }
        
          stage("TRIVY"){
                    steps{
                        sh "trivy image satyadockerhub07/netflix:tagname > trivyimage.txt" 
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
