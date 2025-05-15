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

        stage('SonarQube Scan') {
            steps {
                withSonarQubeEnv('sonar-server') {
                sh """
                    ${tool 'sonar-scanner'}/bin/sonar-scanner \
                    -Dsonar.projectKey=votingapp-service1-vote \
                    -Dsonar.sources=. \
                    -Dsonar.inclusions=**/*.py,**/*.html,**/*.css \
                    -Dsonar.python.version=3
                    """
            }
            }
        }


        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    script {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline failed due to SonarQube Quality Gate status: ${qg.status}"
                        }
                    }
                }
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

      
        stage('Update Manifest') {
            steps {
               sh "sed -i 's|image: heyanoop/voteapp:.*|image: heyanoop/voteapp:${BUILD_NUMBER}|' k8s-specifications/vote-deployment.yaml"
            }
        }


      stage('Deploy to AKS') {
            steps {
                withKubeConfig([serverUrl: "https://391CBA09534E8ADF12782D5FF0588889.gr7.ap-south-1.eks.amazonaws.com", credentialsId: 'cluster-token']) {
                    sh 'kubectl apply -f k8s-specifications/vote-deployment.yaml -n votingapp'
                }
            }
        }
    }
}
