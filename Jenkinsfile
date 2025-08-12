def buildTag = ''

def buildDockerImage(tag) {
    withCredentials([usernamePassword(credentialsId: 'docker-credentails', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
        sh """
            docker build -t sampleapp:${tag} .
            echo "${DOCKER_PASS}" | docker login -u "${DOCKER_USER}" --password-stdin
            docker tag sampleapp:${tag} ${DOCKER_USER}/sampleapp:${tag}
            docker push ${DOCKER_USER}/sampleapp:${tag}
        """
    }
}

pipeline {
    agent { label 'built_agent' }

    parameters {
        string(name: 'NAMESPACE', defaultValue: 'default', description: 'Kubernetes namespace to deploy to')
    }
    
    environment {
        NS = "${params.NAMESPACE}"
    }

    stages {
        stage('Generate Tag') {
            steps {
                script {
                    def date = new Date().format('yyyyMMdd')
                    buildTag = "${date}.${env.BUILD_NUMBER}"
                    env.BUILD_TAG = buildTag
                    currentBuild.displayName = buildTag
                }
            }
        }

        stage('Use Tag') {
            steps {
                script {
                    echo "The build tag is: ${env.BUILD_TAG}"
                }
            }
        }

        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/abhin/Assignment-5-Jenkins-sample-app.git', branch: 'master'
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                script {
                    buildDockerImage(env.BUILD_TAG)
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh """
                        kubectl get namespace ${NS} || kubectl create namespace ${NS}
                        sed -i "s/IMAGE_TAG/${env.BUILD_TAG}/g" deployment.yaml
                        kubectl apply -f deployment.yaml --namespace=${NS}
                    """
                }
            }
        }
    }
}
