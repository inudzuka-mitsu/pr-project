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
                kubectl apply -f pod.yaml
                '''

                sh '''
                echo "Applying service.yaml..."
                kubectl apply -f service.yaml
                '''

                sh '''
                echo "Applying ingress.yaml..."
                kubectl apply -f ingress.yaml
                '''
            }
        }

        stage('Verify Resources') {
            steps {
                sh '''
                echo "Verifying Kubernetes resources..."
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
            kubectl describe pods
            kubectl describe svc
            kubectl describe ingress
            '''
        }
    }
}
