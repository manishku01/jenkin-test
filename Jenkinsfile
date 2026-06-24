pipeline {
    agent any

    tools {
        nodejs 'node js'
    }

    environment {
        CI = 'true'
        S3_BUCKET = 'your-s3-bucket-name'
        AWS_REGION = 'us-east-1'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/manishku01/jenkin-test.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm ci'
            }
        }

        stage('Lint') {
            steps {
                sh 'npm run lint'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test -- --watchAll=false --passWithNoTests'
            }
        }

        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Deploy to S3') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-credentials'
                ]]) {
                    sh '''
                        aws s3 sync build/ s3://$S3_BUCKET \
                            --region $AWS_REGION \
                            --delete \
                            --cache-control "max-age=31536000"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Deployed successfully to s3://${env.S3_BUCKET}"
        }
        failure {
            echo "Build failed. Check console output for details."
        }
    }
}
