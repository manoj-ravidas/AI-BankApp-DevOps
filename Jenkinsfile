pipeline {
agent any

```
environment {
    IMAGE_NAME = "manoj289/ai-bankapp"
    IMAGE_TAG  = "${BUILD_NUMBER}"
    KUBECONFIG = "/var/jenkins_home/.kube/config"
}

stages {

    stage('Checkout') {
        steps {
            checkout scm
        }
    }

    stage('Build Docker Image') {
        steps {
            sh '''
            docker build -t $IMAGE_NAME:$IMAGE_TAG .
            '''
        }
    }

    stage('Docker Login') {
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

    stage('Push Image') {
        steps {
            sh '''
            docker push $IMAGE_NAME:$IMAGE_TAG

            docker tag $IMAGE_NAME:$IMAGE_TAG $IMAGE_NAME:latest

            docker push $IMAGE_NAME:latest
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

            kubectl --kubeconfig=$KUBECONFIG rollout status deployment ai-bankapp --timeout=300s
            '''
        }
    }

    stage('Verify Deployment') {
        steps {
            sh '''
            kubectl --kubeconfig=$KUBECONFIG get nodes

            kubectl --kubeconfig=$KUBECONFIG get pods -o wide

            kubectl --kubeconfig=$KUBECONFIG get svc
            '''
        }
    }
}

post {
    success {
        echo 'Deployment Successful'
    }

    failure {
        echo 'Pipeline Failed'
    }

    always {
        cleanWs()
    }
}
```

}

