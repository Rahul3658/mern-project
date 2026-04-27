pipeline {
    agent any

    environment {
        SONAR_PROJECT = "Mern-stack"
    }

    stages {

        stage('git clone') {
            steps {
                git branch: "$BRANCH",
                url: "https://github.com/Rahul3658/mern-project.git"
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
                            // Pull Request Scan
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
                            // Normal Branch Scan
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

        // stage('SonarQube Analysis') {
        //     steps {
        //         script {
        //             def scannerHome  = tool 'sonar-scanner'
        //             def SONAR_PROJECT = "${SONAR_PROJECT}-${BRANCH}"

        //             withSonarQubeEnv('sonar') {
        //                 sh """
        //                 ${scannerHome}/bin/sonar-scanner \
        //                 -Dsonar.projectKey=${SONAR_PROJECT} \
        //                 -Dsonar.projectName=${SONAR_PROJECT} \
        //                 -Dsonar.branch.name=${BRANCH} \
        //                 -Dsonar.sources=./backend,./frontend \
        //                 -Dsonar.exclusions=node_modules/**,build/**,dist/** \
        //                 -Dsonar.sourceEncoding=UTF-8 \
        //                 -Dsonar.scm.provider=git
        //                 """
        //             }
        //         }
        //     }
        // }
        
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

                    echo "Running Frontend Scan..."
                    cd frontend
                    snyk test --severity-threshold=high || true
                    snyk monitor --project-name=mern-frontend

                    echo "Running Backend Scan..."
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
                to: 'rahul.chaudhari@focalworks.in',
                subject: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
Build Status : SUCCESS
Project      : ${env.JOB_NAME}
Build Number : ${env.BUILD_NUMBER}

SonarQube Scan Completed Successfully.

Dashboard:
http://192.168.7.156:9000/dashboard?id=${SONAR_PROJECT}

Build URL:
${env.BUILD_URL}
                """
            )
        }

        failure {
            emailext(
                to: 'rahul.chaudhari@focalworks.in',
                subject: "FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
Build Status : FAILED
Project      : ${env.JOB_NAME}
Build Number : ${env.BUILD_NUMBER}

Please check Jenkins logs or SonarQube Quality Gate.

Dashboard:
http://192.168.7.156:9000/dashboard?id=${SONAR_PROJECT}

Build URL:
${env.BUILD_URL}
                """
            )
        }
    }
}
