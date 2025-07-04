pipeline {
    agent any

    stages {
        stage('Get Code') {
            steps {
                cleanWs()
                script {
                    def branch = env.GIT_BRANCH?.replaceFirst(/^origin\//, '')
                    git branch: branch, url: 'https://github.com/Carlitos-Etobar/todo-list-aws.git'

                    sh '''
                        rm -f samconfig.toml
                        if [ "${GIT_BRANCH}" = "origin/master" ]; then
                            git clone --branch production https://github.com/Carlitos-Etobar/todo-list-aws-config.git config-repo
                        else
                            git clone --branch staging https://github.com/Carlitos-Etobar/todo-list-aws-config.git config-repo
                        fi
                        cp config-repo/samconfig.toml .
                        rm -rf config-repo
                    '''
                }
            }
        }

        stage('Static Test') {
            when {
                expression { env.BRANCH_NAME == 'develop' }
            }
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
                    sam deploy || [ $? -eq 1 ]
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

        stage('Promote') {
            when {
                allOf {
                    expression { env.BRANCH_NAME == 'develop' }
                    expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' }
                }
            }
            steps {
                sh '''
                    git fetch origin
                    git checkout master || git checkout -b master origin/master
                    git reset --hard
                    git pull origin master --rebase

                    git checkout origin/develop -- .

                    git reset HEAD Jenkinsfile_agentes
                    git restore --staged Jenkinsfile_agentes
                    git restore Jenkinsfile_agentes

                    git commit -m "merge develop into master"
                    git push origin master
                '''
            }
        }
    }
}
