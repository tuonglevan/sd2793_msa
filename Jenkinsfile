def buildDockerImage(def imageName, def imageVersion, def path) {
    sh "docker build -t ${imageName}:ver-${imageVersion} ${path}"
}

def uploadDockerImageToECR(def awsRegion, def ecrURI, def imageName) {
    sh "aws ecr get-login-password --region ${awsRegion} | docker login --username AWS --password-stdin ${ecrURI}"
    sh "docker tag ${imageName}:ver-${BUILD_ID} ${ecrURI}/${imageName}:ver-${BUILD_ID}"
    sh "docker push ${ecrURI}/${imageName}:ver-${BUILD_ID}"
    // Latest version
    sh "docker tag ${imageName}:ver-${BUILD_ID} ${ecrURI}/${imageName}:latest"
    sh "docker push ${ecrURI}/${imageName}:latest"
}

pipeline {
    agent none
    environment {
        REGION = "ap-southeast-1"
        ECR_URI = "010438499500.dkr.ecr.${REGION}.amazonaws.com"
        ECR_BACKEND_IMAGE_NAME = "backend"
        ECR_FRONTED_IMAGE_NAME = "frontend"
    }
    stages {
        stage('Build Backend Image') {
            agent any
            steps {
                script {
                    buildDockerImage(ECR_BACKEND_IMAGE_NAME, BUILD_ID, './src/backend')
                }
            }
        }
        stage('Build Frontend Image') {
            agent any
            steps {
                script {
                    buildDockerImage(ECR_FRONTED_IMAGE_NAME, BUILD_ID, './src/frontend')
                }
            }
        }
        stage('Upload Backend Image to ECR') {
            agent any
            steps {
                script {
                    uploadDockerImageToECR(REGION, ECR_URI, ECR_BACKEND_IMAGE_NAME)
                }
            }
        }
        stage('Upload Frontend Image to ECR') {
            agent any
            steps {
                script {
                  uploadDockerImageToECR(REGION, ECR_URI, ECR_FRONTED_IMAGE_NAME)
                }
            }
       }
    }
}