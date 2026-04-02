pipeline {
    agent any

    environment {
        DOCKER_USERNAME = 'sumanthtony'
        IMAGE_TAG       = "${env.BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    // Frontend
                    sh "docker build -t ${DOCKER_USERNAME}/frontend:${IMAGE_TAG} src/frontend"

                    // Accounts group
                    sh "docker build -t ${DOCKER_USERNAME}/userservice:${IMAGE_TAG} src/accounts/userservice"
                    sh "docker build -t ${DOCKER_USERNAME}/contacts:${IMAGE_TAG} src/accounts/contacts"

                    // Ledger group
                    sh "docker build -t ${DOCKER_USERNAME}/ledgerwriter:${IMAGE_TAG} src/ledger/ledgerwriter"
                    sh "docker build -t ${DOCKER_USERNAME}/balancereader:${IMAGE_TAG} src/ledger/balancereader"
                    sh "docker build -t ${DOCKER_USERNAME}/transactionhistory:${IMAGE_TAG} src/ledger/transactionhistory"

                    // Load generator
                    sh "docker build -t ${DOCKER_USERNAME}/loadgenerator:${IMAGE_TAG} src/loadgenerator"
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DH_USER',
                    passwordVariable: 'DH_PASS'
                )]) {
                    sh 'echo $DH_PASS | docker login -u $DH_USER --password-stdin'
                    script {
                        def services = [
                            'frontend',
                            'userservice',
                            'contacts',
                            'ledgerwriter',
                            'balancereader',
                            'transactionhistory',
                            'loadgenerator'
                        ]
                        services.each { svc ->
                            sh "docker push ${DOCKER_USERNAME}/${svc}:${IMAGE_TAG}"
                            // Also tag as latest
                            sh "docker tag ${DOCKER_USERNAME}/${svc}:${IMAGE_TAG} ${DOCKER_USERNAME}/${svc}:latest"
                            sh "docker push ${DOCKER_USERNAME}/${svc}:latest"
                        }
                    }
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                script {
                    def services = [
                        'frontend',
                        'userservice',
                        'contacts',
                        'ledgerwriter',
                        'balancereader',
                        'transactionhistory',
                        'loadgenerator'
                    ]
                    services.each { svc ->
                        sh "kubectl set image deployment/${svc} ${svc}=${DOCKER_USERNAME}/${svc}:${IMAGE_TAG}"
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Pipeline failed — check logs above'
        }
        always {
            sh 'docker logout'
        }
    }
}
