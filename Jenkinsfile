pipeline {
    agent {
        node {
            label 'agent01'
        }
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/inudzuka-mitsu/pr-project'
            }
        }

        stage('Create Kubernetes Resources') {
            steps {
                sh '/usr/local/bin/kubectl apply -f pod.yaml'
                sh '/usr/local/bin/kubectl apply -f service.yaml'
                sh '/usr/local/bin/kubectl apply -f ingress.yaml'
            }
        }

        stage('Verify Resources') {
            steps {
                sh '/usr/local/bin/kubectl get pods'
                sh '/usr/local/bin/kubectl get svc'
                sh '/usr/local/bin/kubectl get ingress'
            }
        }

        stage('Auto Test') {
            steps {
                script {
                    def serviceHost = "nginx-hello.local"
                    def externalIP = "35.188.208.88"

                    echo "Testing service at: ${serviceHost} (IP: ${externalIP})"

                    def response = sh(
                        script: "curl -s -H 'Host: ${serviceHost}' http://${externalIP}",
                        returnStdout: true
                    ).trim()

                    echo "Response: ${response}"

                    if (response.contains("Hello World!")) { 
                        echo "Application is working as expected!"
                    } else {
                        error "Application did not return the expected response."
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Kubernetes resources created successfully!'
        }
        failure {
            echo 'Failed to create Kubernetes resources.'
            sh '/usr/local/bin/kubectl describe pods'
            sh '/usr/local/bin/kubectl describe svc'
            sh '/usr/local/bin/kubectl describe ingress'
        }
    }
}
