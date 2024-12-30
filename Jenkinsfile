pipeline {
    agent {
        node {
            label 'agent01'
        }
    }
    
    environment {
        KUBECONFIG = credentials('kubeconfig') 
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
                    def testRepo = 'https://github.com/inudzuka-mitsu/cypress-tests'
                    def testBranch = 'master'
                    def testDir = 'cypress-tests'

                    echo "Cloning Cypress test repository: ${testRepo}"
                    sh """
                    rm -rf ${testDir}
                    git clone -b ${testBranch} ${testRepo} ${testDir}
                    cd ${testDir}
                    npm install
                    """

                    echo "Running Cypress tests"
                    sh """
                    cd ${testDir}
                    CYPRESS_BROWSER_ARGS="--disable-gpu" npx cypress run --headless
                    """
                }
            }
        }

        // stage('Clean Up Resources') {
        //     when {
        //         expression {
        //             currentBuild.result == null
        //         }
        //     }
        //     steps {
        //         echo "Deleting Kubernetes resources..."
        //         sh '''
        //         /usr/local/bin/kubectl delete -f ingress.yaml
        //         /usr/local/bin/kubectl delete -f service.yaml
        //         /usr/local/bin/kubectl delete -f pod.yaml
        //         '''
        //     }
        // }
    }

    post {
        success {
            echo 'Pipeline executed successfully and resources were deleted!'
        }
        failure {
            echo 'Pipeline failed. Resources will not be deleted automatically.'
            sh '/usr/local/bin/kubectl describe pods'
            sh '/usr/local/bin/kubectl describe svc'
            sh '/usr/local/bin/kubectl describe ingress'
        }
        always {
            cleanWs()
        }
    }
}
