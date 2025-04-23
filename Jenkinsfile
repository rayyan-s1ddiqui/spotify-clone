pipeline {
    agent any

    environment {
        GIT_REPO_URL       = 'https://github.com/rayyan-s1ddiqui/spotify-clone.git'
        DOCKER_IMAGE_NAME  = 'spotify-clone'
        AWS_REGION         = 'us-east-1'
        ECR_REPO_NAME      = 'magento_repo'
        IMAGE_TAG          = 'latest'
        AWS_CREDENTIALS_ID = 'aws-creds' // from Jenkins Credentials Manager
    }

    stages {
        stage('📥 Clone Repository') {
            steps {
                git "${GIT_REPO_URL}"
            }
        }

        stage('🔐 Login to ECR and Get Registry URI') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: "${AWS_CREDENTIALS_ID}"]]) {
                    script {
                        env.ECR_URL = sh(
                            script: "aws ecr describe-repositories --repository-names ${ECR_REPO_NAME} --region ${AWS_REGION} --query 'repositories[0].repositoryUri' --output text",
                            returnStdout: true
                        ).trim()
                    }
                }
            }
        }

        stage('🔧 Build & Push with Kaniko') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: "${AWS_CREDENTIALS_ID}"]]) {
                    container('kaniko') {
                        script {
                            sh '''
                            mkdir -p /kaniko/.aws
                            echo "[default]" > /kaniko/.aws/credentials
                            echo "aws_access_key_id=$AWS_ACCESS_KEY_ID" >> /kaniko/.aws/credentials
                            echo "aws_secret_access_key=$AWS_SECRET_ACCESS_KEY" >> /kaniko/.aws/credentials

                            /kaniko/executor \
                              --context `pwd` \
                              --dockerfile `pwd`/Dockerfile \
                              --destination=${ECR_URL}:${IMAGE_TAG} \
                              --verbosity=info \
                              --reproducible
                            '''
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ Image successfully built and pushed with Kaniko: ${env.ECR_URL}:${IMAGE_TAG}"
        }
        failure {
            echo '❌ Build failed. Check logs for details.'
        }
    }
}
