pipeline {
    agent any

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/inudzuka-mitsu/pr-project'
            }
        }

        stage('Create Kubernetes Resources') {
            steps {
                sh '''
                echo "Applying pod.yaml..."
                export KUBECONFIG=$KUBE_CONFIG
                kubectl apply -f pod.yaml
                '''

                sh '''
                echo "Applying service.yaml..."
                export KUBECONFIG=$KUBE_CONFIG
                kubectl apply -f service.yaml
                '''

                sh '''
                echo "Applying ingress.yaml..."
                export KUBECONFIG=$KUBE_CONFIG
                kubectl apply -f ingress.yaml
                '''
            }
        }

        stage('Verify Resources') {
            steps {
                sh '''
                echo "Verifying Kubernetes resources..."
                export KUBECONFIG=$KUBE_CONFIG
                kubectl get pods
                kubectl get svc
                kubectl get ingress
                '''
            }
        }
    }

    post {
        success {
            echo 'Kubernetes resources created successfully!'
        }
        failure {
            echo 'Failed to create Kubernetes resources.'
            sh '''
            echo "Dumping logs for debugging..."
            export KUBECONFIG=$KUBE_CONFIG
            kubectl describe pods
            kubectl describe svc
            kubectl describe ingress
            '''
        }
    }
}
