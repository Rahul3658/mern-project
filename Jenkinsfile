pipeline {
    
     parameters {
        choice(
            name: 'BRANCH',
            choices: ['main', 'dev', 'feature'],
            description: 'Select Git Branch to Build'
        )
    }
    
    agent {
        node {
            label 'MERN'
            customWorkspace "/mnt/vol/jenkins/${env.JOB_NAME}"
        }
    }

    tools {
        nodejs '20.0.0'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
        DOCKER_IMAGE_BACKEND = "Rahul3658/mern-backend"
        DOCKER_IMAGE_FRONTEND = "Rahul3658/mern-frontend"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: "${params.BRANCH}",
                    credentialsId: 'AI-Github-Jenkins',
                    url: 'https://github.com/Rahul3658/mern-project.git'
            }
        }
        
           /* ---------------------------
             SECRETS SCAN - GITLEAKS
        ----------------------------- */
        stage("Install GitLeaks") {
            steps {
                sh '''
                if ! command -v gitleaks >/dev/null 2>&1; then
                    echo "Installing GitLeaks..."

                    LATEST=$(curl -s https://api.github.com/repos/gitleaks/gitleaks/releases/latest \
                             | grep browser_download_url \
                             | grep linux_x64.tar.gz \
                             | cut -d '"' -f 4)

                    wget "$LATEST" -O gitleaks.tar.gz
                    tar -xzf gitleaks.tar.gz

                    BIN=$(find . -type f -name "gitleaks" | head -n 1)

                    chmod +x "$BIN"
                    sudo mv "$BIN" /usr/local/bin/gitleaks
                fi

                gitleaks version
                '''
            }
        }

        stage('GitLeaks Scan') {
            steps {
                sh 'gitleaks detect --source ./frontend --exit-code 1'
                sh 'gitleaks detect --source ./backend --exit-code 1'
            }
        }

        /* ---------------------------
                SAST - SONARQUBE
        ----------------------------- */
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
                            -Dsonar.pullrequest.key=${env.CHANGE_ID} \
                            -Dsonar.pullrequest.branch=${env.CHANGE_BRANCH} \
                            -Dsonar.pullrequest.base=${env.CHANGE_TARGET} \
                            -Dsonar.sources=./backend,./frontend \
                            -Dsonar.exclusions=node_modules/**,build/**,dist/**
                            """
                        } else {
                            sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=Mern-stack \
                            -Dsonar.projectName=Mern-stack \
                            -Dsonar.branch.name=${BRANCH} \
                            -Dsonar.sources=./backend,./frontend \
                            -Dsonar.exclusions=node_modules/**,build/**,dist/**
                            """
                        }
                    }
                }
            }
        }

        /* ---------------------------
          SCA - OWASP DEPENDENCY CHECK
        ----------------------------- */
        // stage("OWASP Dependency Check") {
        //     steps {
        //         script {
        //             // TODO: if you use a suppression file, put it under repo and update path below.
        //             dependencyCheck additionalArguments: '''
        //                 --scan ./ \
        //                 --out ./dependency-check-report \
        //                 --format XML --format HTML \
        //                 --failOnCVSS 7
        //             ''', odcInstallation: 'dc'

        //             dependencyCheckPublisher pattern: 'dependency-check-report/dependency-check-report.xml'
        //         }
        //     }
        // }

        /* ---------------------------
               SBOM - CYCLONEDX
        ----------------------------- */
        stage('Generate SBOM (CycloneDX)') {
            steps {
                sh '''
                # Install CycloneDX generator (only first time this will take some time)
                npm install -g @cyclonedx/bom || true

                # Backend SBOM
                cd backend
                cyclonedx-bom -o ../sbom-backend.xml || true

                # Frontend SBOM
                cd ../frontend
                cyclonedx-bom -o ../sbom-frontend.xml || true
                '''
            }
        }

         stage('Trivy FS Scan') {
            steps {
                sh 'trivy fs --format table -o fs-report.html .'
            }
        }

        stage('Build Docker Images') {
            steps {
                sh """
                docker build -f backend/Dockerfile -t $DOCKER_IMAGE_BACKEND:$IMAGE_TAG backend
                docker build -f frontend/Dockerfile -t $DOCKER_IMAGE_FRONTEND:$IMAGE_TAG frontend
                """
            }
        }
        
        
        /* ---------------------------
             TRIVY IMAGE SCAN ✅
        ----------------------------- */
       stage('Trivy Image Scan') {
    steps {
        sh """
        trivy image --severity HIGH,CRITICAL --ignore-unfixed --no-progress $DOCKER_IMAGE_BACKEND:$IMAGE_TAG
        trivy image --severity HIGH,CRITICAL --ignore-unfixed --no-progress $DOCKER_IMAGE_FRONTEND:$IMAGE_TAG
        """
    }
}

        stage('Push to DockerHub') {
            steps {
                sh """
                echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                docker push $DOCKER_IMAGE_BACKEND:$IMAGE_TAG
                docker push $DOCKER_IMAGE_FRONTEND:$IMAGE_TAG
                """
            }
        }

        stage('Deploy Containers') {
            steps {
                sh """
                # Pull latest images
                docker pull $DOCKER_IMAGE_BACKEND:$IMAGE_TAG
                docker pull $DOCKER_IMAGE_FRONTEND:$IMAGE_TAG

                # Stop & remove old containers
                docker stop mern-backend || true
                docker rm mern-backend || true

                docker stop mern-frontend || true
                docker rm mern-frontend || true

                # Run Backend
                docker run -d \
                  --name mern-backend \
                  --restart always \
                  -p 5000:5000 \
                  $DOCKER_IMAGE_BACKEND:$IMAGE_TAG

                # Run Frontend
                docker run -d \
                  --name mern-frontend \
                  --restart always \
                  -p 3000:80 \
                  $DOCKER_IMAGE_FRONTEND:$IMAGE_TAG
                """
            }
        }
    }

    post {
        success {
            echo "Build, Push & Deployment Successful 🚀"
        }
        failure {
            echo "Pipeline Failed ❌"
        }
    }
}
