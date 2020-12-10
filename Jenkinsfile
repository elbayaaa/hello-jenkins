pipeline {
    agent {
        docker {
            args '-u root:sudo'
        }
    }
    stages {
        stage('Install sam-cli') {
            steps {
                sh 'sudo apt-get install python3-venv'
                sh 'python3 -m venv venv && venv/bin/pip install aws-sam-cli'
                stash includes: '**/venv/**/*', name: 'venv'
            }
        }
        stage('Build') {
            steps {
                unstash 'venv'
                sh 'venv/bin/sam build'
                stash includes: '**/.aws-sam/**/*', name: 'aws-sam'
            }
        }
        stage('beta') {
            environment {
                STACK_NAME = 'sam-app-beta-stage'
                S3_BUCKET = 'elbayaaa-jenkins-demo-us-west-2'
            }
            steps {
                withAWS(credentials: 'sam-jenkins-demo-credentials', region: 'us-west-2') {
                    unstash 'venv'
                    unstash 'aws-sam'
                    sh 'venv/bin/sam deploy --stack-name $STACK_NAME -t template.yaml --s3-bucket $S3_BUCKET --capabilities CAPABILITY_IAM'
                    dir ('hello-world') {
                        sh 'npm ci'
                        sh 'npm run integ-test'
                    }
                }
            }
        }
        stage('prod') {
            environment {
                STACK_NAME = 'sam-app-prod-stage'
                S3_BUCKET = 'elbayaaa-jenkins-demo-us-east-1'
            }
            steps {
                withAWS(credentials: 'sam-jenkins-demo-credentials', region: 'us-east-1') {
                    unstash 'venv'
                    unstash 'aws-sam'
                    sh 'venv/bin/sam deploy --stack-name $STACK_NAME -t template.yaml --s3-bucket $S3_BUCKET --capabilities CAPABILITY_IAM'
                }
            }
        }
    }
}
