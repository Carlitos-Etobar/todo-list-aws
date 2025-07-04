pipeline {
    agent any

    stages {
        stage('Get Code') {
            steps {
                cleanWs()
                git branch: 'master', url: 'https://github.com/Carlitos-Etobar/todo-list-aws.git'

                sh '''
                    git clone --branch production --depth 1 https://github.com/Carlitos-Etobar/todo-list-aws-config.git config-repo
                    cp config-repo/samconfig.toml .
                    rm -rf config-repo
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    sam build
                    sam deploy
                '''
            }
        }

        stage('Rest Test') {
            steps {
                sh '''
                    STACK_NAME=$(awk -F' *= *' '/stack_name/ {print $2}' samconfig.toml | tr -d '"')
                    export BASE_URL=$(aws cloudformation describe-stacks --stack-name $STACK_NAME --query "Stacks[0].Outputs[?OutputKey=='BaseUrlApi'].OutputValue" --output text)
                    python3 -m pytest -k "get or list" test/integration/todoApiTest.py || exit 1
                '''
            }
        }
    }
}
