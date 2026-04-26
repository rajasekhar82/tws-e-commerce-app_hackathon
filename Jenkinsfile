pipeline {
    agent any

    environment {
        DOCKER_IMAGE_NAME = 'raja9949/easyshop-app'
        DOCKER_MIGRATION_IMAGE_NAME = 'raja9949/easyshop-migration'
        DOCKER_IMAGE_TAG = "${BUILD_NUMBER}"
        GIT_BRANCH = "master"
        GIT_REPO = "https://github.com/rajasekhar82/tws-e-commerce-app_hackathon.git"
    }

    stages {

        stage('Cleanup Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Clone Repository') {
            steps {
                git branch: "${GIT_BRANCH}",
                    url: "${GIT_REPO}",
                    credentialsId: "github-credentials"
            }
        }

        stage('Build Docker Images') {
            parallel {

                stage('Build Main App Image') {
                    steps {
                        sh """
                        docker build -t ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} .
                        """
                    }
                }

                stage('Build Migration Image') {
                    steps {
                        sh """
                        docker build -f scripts/Dockerfile.migration \
                        -t ${DOCKER_MIGRATION_IMAGE_NAME}:${DOCKER_IMAGE_TAG} .
                        """
                    }
                }
            }
        }

        stage('Run Unit Tests') {
            steps {
                sh """
                echo "Running tests..."
                # mvn test OR npm test (based on your project)
                """
            }
        }

        stage('Security Scan with Trivy') {
            steps {
                sh """
                trivy image ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}
                """
            }
        }

        stage('Push Docker Images') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-hub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin

                    docker push ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}
                    docker push ${DOCKER_MIGRATION_IMAGE_NAME}:${DOCKER_IMAGE_TAG}
                    """
                }
            }
        }

        stage('Update Kubernetes Manifests') {
            steps {
                sh """
                sed -i 's|IMAGE_TAG|${DOCKER_IMAGE_TAG}|g' kubernetes/*.yaml

                git config user.email "rajasekharn82@gmail.com"
                git config user.name "rajasekhar82"

                git add kubernetes/
                git commit -m "Updated image tag to ${DOCKER_IMAGE_TAG}" || true
                git push origin ${GIT_BRANCH}
                """
            }
        }
    }
}
