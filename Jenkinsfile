pipeline {
agent any


environment {
    IMAGE_NAME = "manoj289/ai-bankapp"
    IMAGE_TAG = "${BUILD_NUMBER}"
    KUBECONFIG = "/var/jenkins_home/.kube/config"
}

stages {

    stage('Checkout Source') {
        steps {
            echo "Checking out source code..."
            checkout scm
        }
    }

    stage('Verify Tools') {
        steps {
            sh '''
            echo "=== Docker Version ==="
            docker --version

            echo "=== Kubernetes Nodes ==="
            kubectl --kubeconfig=$KUBECONFIG get nodes
            '''
        }
    }

    stage('Build Docker Image') {
        steps {
            sh '''
            echo "Building Docker image..."
            docker build -t $IMAGE_NAME:$IMAGE_TAG .
            '''
        }
    }

    stage('DockerHub Login') {
        steps {
            withCredentials([
                usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )
            ]) {
                sh '''
                echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                '''
            }
        }
    }

    stage('Push Docker Image') {
        steps {
            sh '''
            echo "Pushing image..."
            docker push $IMAGE_NAME:$IMAGE_TAG

            docker tag $IMAGE_NAME:$IMAGE_TAG $IMAGE_NAME:latest
            docker push $IMAGE_NAME:latest
            '''
        }
    }
stage('Check Files') {
    steps {
        sh '''
        pwd
        ls -la
        find . -name "*.yaml"
        '''
    }
}

    stage('Deploy MySQL') {
        steps {
            sh '''
            kubectl --kubeconfig=$KUBECONFIG apply -f k8s/mysql.yaml
            '''
        }
    }

    stage('Deploy Application') {
        steps {
            sh '''
            kubectl --kubeconfig=$KUBECONFIG apply -f k8s/deployment.yaml
            kubectl --kubeconfig=$KUBECONFIG rollout restart deployment ai-bankapp
            '''
        }
    }

    stage('Wait For Rollout') {
        steps {
            sh '''
            kubectl --kubeconfig=$KUBECONFIG rollout status deployment/ai-bankapp --timeout=300s
            '''
        }
    }

    stage('Verify Deployment') {
        steps {
            sh '''
            echo "=== Deployments ==="
            kubectl --kubeconfig=$KUBECONFIG get deploy

            echo "=== Pods ==="
            kubectl --kubeconfig=$KUBECONFIG get pods -o wide

            echo "=== Services ==="
            kubectl --kubeconfig=$KUBECONFIG get svc
            '''
        }
    }
}

post {
    success {
        echo '===================================='
        echo 'Deployment Successful'
        echo '===================================='
    }

    failure {
        echo '===================================='
        echo 'Pipeline Failed'
        echo '===================================='
    }

    always {
        cleanWs()
    }
}


}

