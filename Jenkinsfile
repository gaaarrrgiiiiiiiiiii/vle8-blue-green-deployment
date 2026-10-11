pipeline {
    agent any

    options {
        timestamps()
    }

    stages {
        stage('Validate') {
            steps {
                sh '''
                    kubectl get nodes
                    kubectl get deployments myapp-blue myapp-green
                    kubectl get service myapp-service
                '''
            }
        }

        stage('Deploy Green') {
            steps {
                sh '''
                    kubectl apply -f k8s/deployment-green.yaml
                    kubectl rollout status deployment/myapp-green --timeout=120s
                '''
            }
        }

        stage('Approve Green') {
            steps {
                input message: 'Green deployment is ready. Switch production traffic to Green?',
                      ok: 'Promote Green'
            }
        }

        stage('Switch Traffic to Green') {
            steps {
                sh '''
                    kubectl patch service myapp-service \
                      -p '{"spec":{"selector":{"app":"myapp","color":"green"}}}'
                    kubectl get endpoints myapp-service
                '''
            }
        }

        stage('Verify Green') {
            steps {
                sh '''
                    kubectl get service myapp-service -o jsonpath='{.spec.selector}{"\\n"}'
                    kubectl get endpoints myapp-service
                '''
            }
        }
    }

    post {
        failure {
            echo 'Pipeline failed. Restoring traffic to Blue.'
            sh '''
                kubectl patch service myapp-service \
                  -p '{"spec":{"selector":{"app":"myapp","color":"blue"}}}' || true
            '''
        }
    }
}
