pipeline {

    agent { label 'slave1' }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerloginid')
    }

    stages {
        stage('SCM Checkout') {
            steps {
                echo '📦 Cloning Insure Me project source code...'
                git 'https://github.com/AbhishekPande285/star-agile-insurance-project.git'
            }
        }

        stage('Application Build') {
            steps {
                echo '🔧 Building the Java Application...'
                sh 'mvn clean package'
            }
        }

        stage('Docker Build') {
            steps {
                echo '🐳 Building Docker Image...'
                sh "docker build -t abhishekpande285/insureme-app:${BUILD_NUMBER} ."
                sh "docker tag abhishekpande285/insureme-app:${BUILD_NUMBER} abhishekpande285/insureme-app:latest"
                sh 'docker image ls'
            }
        }

        stage('Login to DockerHub') {
            steps {
                echo '🔐 Logging into DockerHub...'
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }

        stage('Publish Docker Image') {
            steps {
                echo '📤 Pushing image to DockerHub...'
                sh "docker push abhishekpande285/insureme-app:latest"
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo '🚀 Deploying app to Kubernetes...'
                script {
                    sshPublisher(publishers: [
                        sshPublisherDesc(
                            configName: 'Kubernetes_Master',
                            transfers: [
                                sshTransfer(
                                    sourceFiles: '*.yaml',
                                    remoteDirectory: '.',
                                    execCommand: 'kubectl apply -f kubernetesdeploy.yaml',
                                    execTimeout: 120000
                                )
                            ]
                        )
                    ])
                }
            }
        }
    }

    post {
        success {
            emailext (
                subject: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                <p>🎉 <strong>Build Successful!</strong></p>
                <p><strong>Job:</strong> ${env.JOB_NAME}</p>
                <p><strong>Build Number:</strong> ${env.BUILD_NUMBER}</p>
                <p>🔗 <a href="${env.BUILD_URL}">Click here to view build logs</a></p>
                """,
                to: 'abhipande285@gmail.com',
                mimeType: 'text/html'
            )
        }
        failure {
            emailext (
                subject: "FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                <p>❌ <strong>Build Failed!</strong></p>
                <p><strong>Job:</strong> ${env.JOB_NAME}</p>
                <p><strong>Build Number:</strong> ${env.BUILD_NUMBER}</p>
                <p>🔗 <a href="${env.BUILD_URL}">Click here to view build logs</a></p>
                """,
                to: 'abhipande285@gmail.com',
                mimeType: 'text/html'
            )
        }
    }
}
