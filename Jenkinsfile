pipeline {
    agent any
    environment {
        IMAGE = 'gkeapplication'
        TAG = "${BUILD_NUMBER}"
        PROJECT_ID = 'f4yiee-407411'
        CLUSTER_NAME = 'gkecluster'
        LOCATION = 'us-central1-a'
        CREDENTIALS_ID = '52ca32b2-a487-4545-8ce4-4b4eb0762a1b'
        HELM_CHART_PATH = 'swiggy-app/'
        HELM_RELEASE_NAME = 'swiggy'
        HELM_NAMESPACE = 'swiggy'
    }
    stages {
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/abdulfayis681/gke-application.git'
            }
        }
        stage("Docker Build"){
            steps{
                script{
                    withDockerRegistry(credentialsId: '9181b467-36db-4aea-bc62-816143482973', toolName: 'docker'){   
                        sh "sudo docker build -t ${IMAGE} ."
                    }
                }
            }
        }
        stage(' Trivy Scan') {
            steps {
                //  trivy output template 
                sh 'sudo curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/html.tpl > html.tpl'

                // Scan all vuln levels
                sh 'sudo mkdir -p reports'
                sh 'sudo trivy image --format template --template @./html.tpl -o reports/trivy-report.html ${IMAGE}:latest'
                publishHTML target : [
                    allowMissing: true,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'reports',
                    reportFiles: 'trivy-report.html',
                    reportName: 'Trivy Scan',
                    reportTitles: 'Trivy Scan'
                ]

                // Scan again and fail on CRITICAL vulns
                // sh 'trivy image --ignore-unfixed --vuln-type os,library --exit-code 1 --severity CRITICAL abdulfayis/gkeapplication:latest '

            }
        }
        stage("Docker Push"){
            steps{
                script{
                    withDockerRegistry(credentialsId: '9181b467-36db-4aea-bc62-816143482973', toolName: 'docker'){   
                        sh "sudo docker tag ${IMAGE} abdulfayis/${IMAGE}:${TAG} "
                        sh "sudo docker push abdulfayis/${IMAGE}:${TAG} "
                        sh "sudo docker tag ${IMAGE} abdulfayis/${IMAGE}:latest "
                        sh "sudo docker push abdulfayis/${IMAGE}:latest "
                    }
                }
            }
        }
        stage("Docker Clean up "){
            steps{
                 sh 'echo " cleaning Docker Images"'
                 sh 'sudo docker rmi -f \$(sudo docker images -q)'
            }
        }
        stage("Helm install "){
            steps{
                 sh "curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3"
                 sh "chmod 700 get_helm.sh"
                 sh "./get_helm.sh"
            }
        }
        stage('GKE Authentication') {
            steps{
                    sh "gcloud container clusters get-credentials ${CLUSTER_NAME} --zone ${LOCATION} --project ${PROJECT_ID}"
                
            }
        }
        stage('Deploy Helm Chart') {
            steps {
                script {
                    sh 'kubectl create ns ${HELM_NAMESPACE} || echo "namespace already created" '
                    sh "helm upgrade --install ${HELM_RELEASE_NAME} ${HELM_CHART_PATH} --set image.tag=${TAG} --namespace=${HELM_NAMESPACE} --wait"
                }
            }
        }
        
    }
}
