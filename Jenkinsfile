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
                    def namespace = "default"
                    def serviceHost = "nginx-hello.local"
                    def externalIP = "<EXTERNAL-IP>"

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
