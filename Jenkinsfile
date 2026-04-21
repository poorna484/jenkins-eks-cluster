pipeline {
    agent any

    environment {
        AWS_REGION   = 'us-east-1'
        CLUSTER_NAME = 'my-eks-cluster'
        KUBECONFIG   = "${HOME}/.kube/config"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/poorna484/jenkins-eks-cluster.git'
            }
        }

        stage('Verify Tools') {
            steps {
                sh '''
                    set -e
                    whoami
                    pwd
                    echo "HOME=$HOME"
                    aws --version
                    kubectl version --client
                    eksctl version
                    aws sts get-caller-identity
                '''
            }
        }

        stage('Create EKS Cluster') {
            steps {
                sh '''
                    set -e
                    eksctl create cluster -f cluster-config.yaml
                '''
            }
        }

        stage('Update kubeconfig') {
            steps {
                sh '''
                    set -e
                    mkdir -p $HOME/.kube
                    aws eks update-kubeconfig \
                      --region ${AWS_REGION} \
                      --name ${CLUSTER_NAME} \
                      --kubeconfig ${KUBECONFIG}
                '''
            }
        }

        stage('Validate Cluster') {
            steps {
                sh '''
                    set -e
                    export KUBECONFIG=${KUBECONFIG}
                    kubectl get nodes
                    kubectl get pods -A
                '''
            }
        }
    }

    post {
        success {
            echo 'EKS cluster created and validated successfully.'
        }
        failure {
            echo 'Pipeline failed. Check console output.'
        }
    }
}
