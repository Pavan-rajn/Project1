pipeline {
    agent { label "built-in"}

    environment {
        DOCKERHUB_CREDENTIALS = credentials('0e2b1a7e-86b1-472b-ae24-828e0d2a2aec')
        IMAGE_NAME = "myapp"
        REGISTRY = "pavanrajnikam/myapp1"
        IMAGE_TAG = "1.4"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
                echo "Branch detected: ${env.BRANCH_NAME}"
            }
        }

        stage('Build & Push Docker Image') {
            when {
                anyOf {
                    branch 'master'
                    branch 'develop'
                }
            }
            steps {
                sh """
                    docker build -t ${REGISTRY}:${IMAGE_TAG} .
                    echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                    docker push ${REGISTRY}:${IMAGE_TAG}
                """
            }
        }

        stage('Deploy') {
            when {
                anyOf {
                    branch 'master'
                    branch 'develop'
                }
            }
            steps {
                script {
                    if (env.BRANCH_NAME == 'master') {
                        deployToNode('test', 8081)
                        deployToNode('prod', 8081)
                    } else if (env.BRANCH_NAME == 'develop') {
                        deployToNode('test', 8082)
                    }
                }
            }
        }
    }
}

def deployToNode(nodeName, port) {
    node(nodeName) {
        stage("Deploy to ${nodeName} on port ${port}") {
            sh """
                set -e

                docker pull ${REGISTRY}:${IMAGE_TAG}

                docker ps -q --filter "name=${IMAGE_NAME}" | xargs -r docker stop
                docker ps -aq --filter "name=${IMAGE_NAME}" | xargs -r docker rm

                docker ps -q --filter "publish=${port}" | xargs -r docker stop
                docker ps -aq --filter "publish=${port}" | xargs -r docker rm

                docker system prune -f

                docker run -d -p ${port}:80 --name ${IMAGE_NAME} ${REGISTRY}:${IMAGE_TAG}
            """
        }
    }
}
