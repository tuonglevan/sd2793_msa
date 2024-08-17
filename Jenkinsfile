def buildDockerImage(def imageName, def imageVersion, def path) {
    sh "docker build -t ${imageName}:ver-${imageVersion} ${path}"
}

def uploadDockerImageToECR(def awsRegion, def ecrURI, def imageName, def buildId) {
    def imageTag = "ver-${buildId}"
    def fullTag = "${ecrURI}/${imageName}:${imageTag}"
    def latestTag = "${ecrURI}/${imageName}:latest"

    sh "aws ecr get-login-password --region ${awsRegion} | docker login --username AWS --password-stdin ${ecrURI}"
    sh "docker tag ${imageName}:${imageTag} ${fullTag}"
    sh "docker push ${fullTag}"
    sh "docker tag ${imageName}:${imageTag} ${latestTag}"
    sh "docker push ${latestTag}"
}

pipeline {
    agent none
    environment {
        REGION = "ap-southeast-1"
        AWS_CREDENTIALS_NAME = "aws_devops_credentials"
        ECR_URI = "010438499500.dkr.ecr.${REGION}.amazonaws.com"
        ECR_BACKEND_IMAGE_NAME = "backend"
        ECR_FRONTED_IMAGE_NAME = "frontend"
        EKS_CLUSTER_NAME = "sd2793-devops-eks-cluster"
    }
    stages {
        stage('Build Backend Image') {
            agent any
            steps {
                script {
                    buildDockerImage(env.ECR_BACKEND_IMAGE_NAME, env.BUILD_ID, './src/backend')
                }
            }
        }
        stage('Build Frontend Image') {
            agent any
            steps {
                script {
                    buildDockerImage(env.ECR_FRONTED_IMAGE_NAME, env.BUILD_ID, './src/frontend')
                }
            }
        }
        stage('Upload Backend Image to ECR') {
            agent any
            steps {
                script {
                    uploadDockerImageToECR(env.REGION, env.ECR_URI, env.ECR_BACKEND_IMAGE_NAME, env.BUILD_ID)
                }
            }
        }
        stage('Upload Frontend Image to ECR') {
            agent any
            steps {
                script {
                  uploadDockerImageToECR(env.REGION, env.ECR_URI, env.ECR_FRONTED_IMAGE_NAME, env.BUILD_ID)
                }
            }
       }
       stage('Deploy to EkS') {
            agent any
            steps {
                withAWS(region: env.REGION, credentials: env.AWS_CREDENTIALS_NAME) {
                    sh '''
                       #!/bin/bash
                       set -ex

                       # Debug information
                       echo "Using AWS region: ${env.REGION}"
                       echo "EKS Cluster name: ${env.CLUSTER_NAME}"

                       # Update kubeconfig
                       aws eks update-kubeconfig --name ${env.EKS_CLUSTER_NAME}
                       # Deploy to EKS
                       kubectl apply -f k8s/aws/mongodb.yaml
                       kubectl apply -f k8s/aws/backend.yaml
                       kubectl apply -f k8s/aws/frontend.yaml
                    '''
                }
            }
       }
    }
}