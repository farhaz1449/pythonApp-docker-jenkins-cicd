// pipeline {
//     agent any

//     environment {
//         IMAGE_NAME = "myapp"
//         IMAGE_TAG = "${env.BUILD_NUMBER}"
//     }

//     stages {
        
//         stage('Build Docker Image') {
//             steps {
//                 script {
//                     //dockerImage = docker.build("myapp:${env.BUILD_NUMBER}")
//                     sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ." 
//                 }
//             }
//         }

//         stage('Run Container') {
//             steps {
//                 script {
//                     //dockerImage.run("-p 5000:5000")
//                     sh "docker run -d -p 5000:5000 --name demo-container ${IMAGE_NAME}:${IMAGE_TAG}" 

//                 }
//             }
//         }
//     }
// }


// Instuction for permission: sudo usermod -aG docker jenkins
// sudo systemctl restart jenkins
// sudo systemctl restart docker

// If Deployed in separate Application server then use below command to connect Jenkins server to Docker server

pipeline {
    agent any

    environment {
        APP_NAME      = 'python-docker-app'
        IMAGE_NAME    = 'farhaz1449/python-docker-app'
        DEPLOY_SERVER = '54.89.204.81'
        DEPLOY_USER   = 'ubuntu'
        DEPLOY_PORT   = '22'
    }

    options {
        timestamps()
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                  docker build -t ${IMAGE_NAME}:${env.BUILD_NUMBER} .
                """
            }
        }

        stage('Push Image to Registry') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                      echo "${DOCKER_PASS}" | docker login -u "${DOCKER_USER}" --password-stdin
                      docker push ${IMAGE_NAME}:${env.BUILD_NUMBER}
                    """
                }
            }
        }

        stage('Deploy to Deployment Server') {
            steps {
                sshagent(credentials: ['deploy-server-ssh']) {
                    sh """
                      ssh -o StrictHostKeyChecking=no -p ${DEPLOY_PORT} ${DEPLOY_USER}@${DEPLOY_SERVER} '
                        docker pull ${IMAGE_NAME}:${env.BUILD_NUMBER} &&
                        docker rm -f ${APP_NAME} || true &&
                        docker run -d --name ${APP_NAME} -p 5000:5000 ${IMAGE_NAME}:${env.BUILD_NUMBER}
                      '
                    """
                }
            }
        }
    }

    post {
        failure {
            echo 'Deployment failed.'
        }
        success {
            echo 'Deployment completed successfully.'
        }
    }
}

