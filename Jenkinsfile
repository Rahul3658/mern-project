pipeline {
    agent any

    environment {
        SONAR_PROJECT = "Mern-stack"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Semgrep Scan') {
            steps {
                sh '''
                docker run --rm \
                -v $(pwd):/src \
                semgrep/semgrep \
                semgrep scan --config=auto /src
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'sonar-scanner'

                    withSonarQubeEnv('sonar') {

                        if (env.CHANGE_ID) {

                            sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=Mern-stack \
                            -Dsonar.projectName=Mern-stack \
                            -Dsonar.sources=backend,frontend \
                            -Dsonar.pullrequest.key=${env.CHANGE_ID} \
                            -Dsonar.pullrequest.branch=${env.CHANGE_BRANCH} \
                            -Dsonar.pullrequest.base=${env.CHANGE_TARGET}
                            """

                        } else {

                            sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=Mern-stack \
                            -Dsonar.projectName=Mern-stack \
                            -Dsonar.sources=backend,frontend \
                            -Dsonar.branch.name=${env.BRANCH_NAME}
                            """
                        }
                    }
                }
            }
        }

        stage('Install Dependency') {
            steps {
                sh '''
                    cd backend
                    npm install

                    cd ../frontend
                    npm install
                '''
            }
        }

        stage('Snyk Security Scan') {
            steps {
                withCredentials([string(credentialsId: 'snyk-token', variable: 'SNYK_TOKEN')]) {
                    sh '''
                    export PATH=/var/lib/jenkins/.npm-global/bin:$PATH
                    export SNYK_TOKEN=$SNYK_TOKEN

                    snyk auth $SNYK_TOKEN

                    cd frontend
                    snyk test --severity-threshold=high || true
                    snyk monitor --project-name=mern-frontend

                    cd ../backend
                    snyk test --severity-threshold=high || true
                    snyk monitor --project-name=mern-backend
                    '''
                }
            }
        }
    }

    post {

        success {
            emailext(
                to: 'rahul3658@gmail.com',
                subject: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
Build Status : SUCCESS
Project      : ${env.JOB_NAME}
Build Number : ${env.BUILD_NUMBER}

Dashboard:
http://192.168.7.156:9000/dashboard?id=${SONAR_PROJECT}

Build URL:
${env.BUILD_URL}
"""
            )
        }

        failure {
            emailext(
                to: 'rahul3658@gmail.com',
                subject: "FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
Build Status : FAILED
Project      : ${env.JOB_NAME}
Build Number : ${env.BUILD_NUMBER}

Dashboard:
http://192.168.7.156:9000/dashboard?id=${SONAR_PROJECT}

Build URL:
${env.BUILD_URL}
"""
            )
        }
    }
}
