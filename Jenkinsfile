pipeline {
    agent {
        node {
            label 'agent01'
        }
    }
    
    environment {
        KUBECONFIG = credentials('kubeconfig') 
        PATH = "${PATH}:/home/ubuntu/google-cloud-sdk/bin"
        USE_GKE_GCLOUD_AUTH_PLUGIN = 'True'
    }

    stages {

        stage('Verify Environment') {
            steps {
                sh 'echo $PATH'
                sh 'which gke-gcloud-auth-plugin || echo "Plugin not found in PATH"'
            }
        }

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/inudzuka-mitsu/pr-project'
            }
        }

        stage('Create Kubernetes Resources') {
            steps {
                sh 'kubectl apply -f pod.yaml'
                sh 'kubectl apply -f service.yaml'
                sh 'kubectl apply -f ingress.yaml'
            }
        }

        stage('Verify Resources') {
            steps {
                sh 'kubectl get pods'
                sh 'kubectl get svc'
                sh 'kubectl get ingress'
            }
        }

        stage('Auto Test') {
            steps {
                script {
                    def namespace = "default"
                    def serviceHost = "nginx-hello.local"
                    def externalIP = "35.188.208.88"
                    echo "Testing service at: ${serviceHost} (IP: ${externalIP})"
                    def response = sh(
                        script: "curl -Is -H \"Host: ${serviceHost}\" http://${externalIP}",
                        returnStdout: true
                    ).trim()
                    echo "Response: ${response}"
                    if (response.contains("HTTP/1.1 200 OK")) {
                        echo "Service is accessible: It's OK"
                    } else {
                        error "Service is not accessible: It's NOT OK"
                    }
                }
            }
        }

        stage('Clean Up Resources') {
            when {
                expression {
                    currentBuild.result == null
                }
            }
            steps {
                echo "Deleting Kubernetes resources..."
                sh '''
                kubectl delete -f ingress.yaml
                kubectl delete -f service.yaml
                kubectl delete -f pod.yaml
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully and resources were deleted!'
        }
        failure {
            echo 'Pipeline failed. Resources will not be deleted automatically.'
            sh 'kubectl describe pods || echo "Failed to describe pods"'
            sh 'kubectl describe svc || echo "Failed to describe services"'
            sh 'kubectl describe ingress || echo "Failed to describe ingress"'
        }
        always {
            cleanWs()
        }
    }
}
