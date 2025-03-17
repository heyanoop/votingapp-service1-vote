pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "heyanoop/voteapp:${BUILD_NUMBER}"
    }

    stages {
        stage('Code Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/heyanoop/votingapp-service1-vote.git'
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                withCredentials([string(credentialsId: 'dockerhub-password', variable: 'DOCKER_PASS')]) {
                    sh """
                    docker login -u heyanoop -p ${DOCKER_PASS}
                    docker build -t ${DOCKER_IMAGE} .
                    docker push ${DOCKER_IMAGE}
                    """
                }
            }
        }

        stage('Trivy Security Scan') {
            steps {
                sh "trivy image --exit-code 0 --severity HIGH,CRITICAL ${DOCKER_IMAGE}"
                sh "trivy image --format table ${DOCKER_IMAGE} | tee trivy_scan.log"

                script {
                    echo "🔍 Security Scan Results:"
                    sh "cat trivy_scan.log"
                }
            }
        }

      
        stage('Update Manifest') {
            steps {
                sh "sed -i 's|image: heyanoop/voteapp:.*|image: ${DOCKER_IMAGE}|' k8s-specifications/vote-deployment.yaml"
            }
        }
 
        // stage('Deploy to Cluster') {
            // steps {
                // sh "kubectl apply -f k8s-specifications/vote-deployment.yaml"
            // }
        // }
    }
}
