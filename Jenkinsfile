pipeline {
    agent {
        kubernetes {
            yamlFile 'agent.yaml'
            retries 2
        }
    }

    environment {
        IMAGE_NAME  = 'janobox/tarea-final'
        IMAGE_TAG   = 'alejandro-cespedes'
        APP_VERSION = '3.0.0'
        NAMESPACE   = 'ns-alejandro-cespedes'
        DEPLOYMENT  = 'app-alejandro-cespedes'
    }

    stages {

        stage('install') {
            steps {
                container('node') {
                    sh '''
                        corepack enable
                        pnpm install --frozen-lockfile
                    '''
                }
            }
        }

        stage('test') {
            steps {
                container('node') {
                    sh '''
                        pnpm test --runInBand
                    '''
                }
            }
        }

        stage('build') {
            steps {
                container('node') {
                    sh '''
                        pnpm run build
                    '''
                }

                container('docker') {
                    sh '''
                        echo "Esperando Docker daemon..."

                        intentos=0
                        until docker info >/dev/null 2>&1; do
                            intentos=$((intentos + 1))

                            if [ "$intentos" -ge 30 ]; then
                                echo "Docker daemon no inicio a tiempo"
                                exit 1
                            fi

                            sleep 2
                        done

                        docker build \
                          -t ${IMAGE_NAME}:${IMAGE_TAG} \
                          -t ${IMAGE_NAME}:${APP_VERSION} \
                          .
                    '''
                }
            }
        }

        stage('push') {
            steps {
                container('docker') {
                    withCredentials([
                        usernamePassword(
                            credentialsId: 'dockerhub-credentials',
                            usernameVariable: 'DOCKERHUB_USER',
                            passwordVariable: 'DOCKERHUB_TOKEN'
                        )
                    ]) {
                        sh '''
                            echo "$DOCKERHUB_TOKEN" | \
                              docker login \
                              --username "$DOCKERHUB_USER" \
                              --password-stdin

                            docker push ${IMAGE_NAME}:${IMAGE_TAG}
                            docker push ${IMAGE_NAME}:${APP_VERSION}

                            docker logout
                        '''
                    }
                }
            }
        }

        stage('deploy') {
            steps {
                container('kubectl') {
                    sh '''
                        kubectl apply -f entrega.yaml

                        kubectl rollout restart \
                          deployment/${DEPLOYMENT} \
                          -n ${NAMESPACE}

                        kubectl rollout status \
                          deployment/${DEPLOYMENT} \
                          -n ${NAMESPACE} \
                          --timeout=120s

                        kubectl get pods \
                          -n ${NAMESPACE}
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline CI/CD completado correctamente.'
        }

        failure {
            echo 'Pipeline CI/CD fallo. Revisar Console Output.'
        }
    }
}
