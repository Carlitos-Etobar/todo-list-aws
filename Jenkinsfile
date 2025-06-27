pipeline {
    agent any

    stages {
        stage('Get Code') {
            steps {
                cleanWs()
                git branch: 'develop', url: 'https://github.com/Carlitos-Etobar/todo-list-aws.git'
            }
        }

        stage('Static Test') {
            steps {
                sh '''
                    flake8 src > flake8-report.txt || true
                    bandit -r src -f custom -o bandit-report.out --msg-template "{abspath}:{line}: [{test_id}] {msg}" || true
                '''
                recordIssues tools: [
                    flake8(pattern: 'flake8-report.txt'),
                    pyLint(name: 'Bandit', pattern: 'bandit-report.out')
                ]
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    sam build
                    sam deploy --stack-name todo-app-staging --s3-bucket aws-sam-cli-managed-default-samclisourcebucket-mgenxqjrm4fs --region us-east-1 --capabilities CAPABILITY_IAM --no-fail-on-empty-changeset
                '''
            }
        }

        stage('Rest Test') {
            steps {
                sh '''
                    export BASE_URL=$(aws cloudformation describe-stacks --stack-name todo-app-staging --query "Stacks[0].Outputs[?OutputKey=='ApiUrl'].OutputValue" --output text)
                    python3 -m pytest test/integration/todoApiTest.py || exit 1
                '''
            }
        }

        stage('Promote') {
            when {
                expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' }
            }
            steps {
                sh '''
                    git fetch origin
                    git checkout master || git checkout -b master origin/master
                    git merge origin/develop
                    git push origin master
                '''
            }
        }
    }
}
