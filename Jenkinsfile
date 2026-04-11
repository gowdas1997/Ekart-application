pipeline {
    agent any

    environment {
        PROJECT_ID      = "leafy-chariot-485812-k6"
        REGION          = "asia-south1"
        REPO            = "ekart-repo"
        IMAGE_NAME      = "ekart-app"
        REGISTRY        = "${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPO}/${IMAGE_NAME}"
        IMAGE_TAG       = "${BUILD_NUMBER}"
        FULL_IMAGE      = "${REGISTRY}:${IMAGE_TAG}"
        K8S_DEPLOYMENT  = "ekart-deployment"
        K8S_NAMESPACE   = "default"
        GKE_CLUSTER     = "ekart-cluster"
        GKE_ZONE        = "asia-south1-a"
    }

    tools {
        maven 'Maven-3.9'
        jdk   'JDK-17'
    }

    stages {

        stage('Checkout') {
            steps {
                echo '📥 Code checkout maadtha idhe...'
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo '🔨 Maven build maadtha idhe...'
                sh 'mvn clean package -DskipTests'
            }
            post {
                success {
                    echo '✅ Build successful!'
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                }
                failure {
                    echo '❌ Build failed!'
                }
            }
        }

        stage('Test') {
            steps {
                echo '🧪 Tests run maadtha idhe...'
                sh 'mvn test'
            }
            post {
                always {
                    junit allowEmptyResults: true,
                          testResults: 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Docker Build') {
            steps {
                echo "🐳 Docker image build maadtha idhe..."
                sh """
                    docker build -t ${FULL_IMAGE} -f docker/Dockerfile .
                    docker tag ${FULL_IMAGE} ${REGISTRY}:latest
                """
            }
        }

        stage('Push to Artifact Registry') {
            steps {
                echo '📤 Artifact Registry ge push maadtha idhe...'
                sh """
                    gcloud auth configure-docker ${REGION}-docker.pkg.dev --quiet
                    docker push ${FULL_IMAGE}
                    docker push ${REGISTRY}:latest
                """
            }
        }

        stage('Deploy to GKE') {
            steps {
                echo '🚀 GKE ge deploy maadtha idhe...'
                sh """
                    gcloud container clusters get-credentials ${GKE_CLUSTER} \
                        --zone=${GKE_ZONE} \
                        --project=${PROJECT_ID}

                    kubectl set image deployment/${K8S_DEPLOYMENT} \
                        ekart-container=${FULL_IMAGE} \
                        -n ${K8S_NAMESPACE}

                    kubectl rollout status deployment/${K8S_DEPLOYMENT} \
                        -n ${K8S_NAMESPACE} --timeout=120s

                    kubectl get pods -n ${K8S_NAMESPACE}
                    kubectl get svc -n ${K8S_NAMESPACE}
                """
            }
            post {
                failure {
                    echo '❌ Deploy failed! Rollback maadtha idhe...'
                    sh """
                        kubectl rollout undo deployment/${K8S_DEPLOYMENT} \
                            -n ${K8S_NAMESPACE}
                    """
                }
            }
        }
    }

    post {
        success {
            echo '🎉 Pipeline complete! Ekart app live aagide!'
        }
        failure {
            echo '💥 Pipeline failed! Logs check maadi.'
        }
        always {
            cleanWs()
        }
    }
}
