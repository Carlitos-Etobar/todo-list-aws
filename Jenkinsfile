pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        STACK_NAME = 'todo-app-production'
        S3_BUCKET = 'aws-sam-cli-managed-default-samclisourcebucket-mgenxqjrm4fs'
    }

    stages {
        stage('Get Code') {
            steps {
                cleanWs()
                git branch: 'master', url: 'https://github.com/Carlitos-Etobar/todo-list-aws.git'
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    sam build
                    sam deploy --stack-name $STACK_NAME --s3-bucket $S3_BUCKET --region $AWS_REGION --capabilities CAPABILITY_IAM --no-fail-on-empty-changeset --parameter-overrides Stage=production
                '''
            }
        }

        stage('Rest Test') {
            steps {
                sh '''
                    export BASE_URL=$(aws cloudformation describe-stacks --stack-name $STACK_NAME --query "Stacks[0].Outputs[?OutputKey=='BaseUrlApi'].OutputValue" --output text)
                    python3 -m pytest -k "get or list" test/integration/todoApiTest.py || exit 1
                '''
            }
        }
    }
}
