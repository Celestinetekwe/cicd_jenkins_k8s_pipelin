pipeline {
    agent any

    environment {
        AWS_ACCESS_KEY_ID = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
        AWS_DEFAULT_REGION = "us-east-1"
        ECR_REPOSITORY = 'sample-app'
        EKS_CLUSTER_NAME = 'dedicated-cluster'
        NAMESPACE = 'default'
        AWS_ACCOUNT_ID = '108286109057'
        IMAGE_TAG = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
        REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${ECR_REPOSITORY}"
    }

    stages {
       /** stage("Create an EKS Cluster") {
            steps {
                script {
                    dir('Terraform') {
                        sh "terraform init"
                        sh "terraform apply -auto-approve"
                    }
                }
            }
        } **/
        stage('Logging into Github') {
            steps {
                script {
                    checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/Celestinetekwe/cicd_jenkins_k8s_pipelin']])
                }
            }
        }

        stage('Building image') {
            steps {
                script {
                    dockerImage = docker.build "${ECR_REPOSITORY}:${IMAGE_TAG}"
                }
            }
        }

        stage('Push to ECR') {
            steps {
                script {
                    sh "aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
                    sh "docker tag ${ECR_REPOSITORY}:${IMAGE_TAG} ${REPOSITORY_URI}:${IMAGE_TAG}"
                    sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${ECR_REPOSITORY}:${IMAGE_TAG}"
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                script {
                    sh "aws eks update-kubeconfig --name ${EKS_CLUSTER_NAME} --region ${AWS_DEFAULT_REGION} --kubeconfig ~/.kube/config"
                    sh "kubectl set image deployment/nginx nginx=${REPOSITORY_URI}:${IMAGE_TAG} -n ${NAMESPACE} --kubeconfig ~/.kube/config"
                    sh "kubectl apply -f kubernetes/service.yaml --kubeconfig ~/.kube/config -n ${NAMESPACE}"
                }
            }
        }
    }

    post {
        always {
            sh "rm /tmp/kubeconfig"
        }
    }
}
