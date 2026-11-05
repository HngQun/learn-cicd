pipeline {
    agent any
    options {
        disableConcurrentBuilds()
        skipDefaultCheckout(true)
        timeout(time: 20, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '20'))
    }
    triggers { pollSCM('H/5 * * * *') }
    stages {
        stage('Checkout main') {
            steps {
                deleteDir()
                checkout scm
                sh 'test "$(git rev-parse HEAD)" = "$(git rev-parse origin/main)"'
                script { env.IMAGE_TAG = sh(script: 'git rev-parse HEAD', returnStdout: true).trim() }
            }
        }
        stage('Validate') {
            steps {
                sh '''
                    set -eu
                    test -s index.html
                    grep -qi '<html' index.html
                    grep -qi '</html>' index.html
                    docker compose -p server-learn-cicd -f compose.yml config --quiet
                '''
            }
        }
        stage('Build') {
            steps { sh 'docker build --pull -t "learn-cicd:$IMAGE_TAG" .' }
        }
        stage('Deploy') {
            steps {
                sh 'docker compose -p server-learn-cicd -f compose.yml up -d --wait --wait-timeout 120'
            }
        }
        stage('Verify') {
            steps {
                sh '''
                    set -eu
                    docker compose -p server-learn-cicd -f compose.yml exec -T web wget -qO- http://127.0.0.1/ > served.html
                    cmp index.html served.html
                '''
            }
        }
    }
    post {
        failure {
            sh 'docker compose -p server-learn-cicd -f compose.yml logs --tail 80 || true'
        }
    }
}
