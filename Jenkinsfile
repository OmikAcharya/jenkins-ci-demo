pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    python3 -m venv venv
                    ./venv/bin/pip install -r requirements.txt
                '''
            }
        }

        stage('Lint') {
    steps {
        sh '''
            echo "=== LINTING ==="
            pwd
            echo "Jenkinsfile says:"
            grep -n flake8 Jenkinsfile
            echo "Running:"
            ./venv/bin/flake8 --exclude=venv app.py test_app.py
        '''
    }
}

        stage('Unit Tests') {
            steps {
                sh './venv/bin/pytest'
            }
        }

        stage('Package') {
            steps {
                sh '''
                    rm -rf artifact
                    mkdir -p artifact
                    cp app.py test_app.py requirements.txt Jenkinsfile artifact/
                    echo "$GIT_COMMIT" > artifact/commit-sha.txt
                    tar -czf "application-${GIT_COMMIT}.tar.gz" -C artifact .
                '''
            }
        }

        stage('Upload to S3') {
            steps {
                sh '''
                    test -n "$S3_BUCKET" || {
                        echo "ERROR: S3_BUCKET environment variable is not set."
                        exit 1
                    }

                    aws s3 cp                         "application-${GIT_COMMIT}.tar.gz"                         "s3://${S3_BUCKET}/"
                '''
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'application-*.tar.gz', allowEmptyArchive: true
        }
    }
}
