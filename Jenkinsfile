pipeline {
    agent { label 'slave1' }

    stages {
        stage('checkout') {
            steps {
                sh 'rm -rf hello-world-war'
                sh 'git clone https://github.com/Roopa2487/hello-world-war.git'
            }
        }
        
        stage('build') {
            steps {
                dir("hello-world-war") {
                    sh 'echo "inside build"'
                    sh "docker build -t tomcat-war:${BUILD_NUMBER} ."
                }
            }
        }
        
        stage('push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dckr_pat_H7R-co42-PaD4GUhMfOcofvdXqo', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh "docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}"
                    sh "docker tag tomcat-war:${BUILD_NUMBER} roopa2487/tomcat:${BUILD_NUMBER}"
                    sh "docker push roopa2487/tomcat:${BUILD_NUMBER}"
                }
            }
        }
        
        stage('deploy') {
            parallel {
                stage('deployQA') {
                    agent { label 'slave2' }
                    steps {
                        script {
                            withCredentials([usernamePassword(credentialsId: 'dckr_pat_H7R-co42-PaD4GUhMfOcofvdXqo', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                                sh "docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}"
                                sh "docker pull roopa2487/tomcat:${BUILD_NUMBER}"
                                sh 'docker rm -f tomcat-qa || true'
                                sh 'docker run -d -p 5555:8080 --name tomcat-qa roopa2487/tomcat:${BUILD_NUMBER}'
                            }
                        }
                    }
                }
                stage('deployProd') {
                    agent { label 'slave3' }
                    steps {
                        script {
                            withCredentials([usernamePassword(credentialsId: 'dckr_pat_H7R-co42-PaD4GUhMfOcofvdXqo', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                                sh "docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}"
                                sh "docker pull roopa2487/tomcat:${BUILD_NUMBER}"
                                sh 'docker rm -f tomcat-prod || true'
                                sh 'docker run -d -p 5555:8080 --name tomcat-prod roopa2487/tomcat:${BUILD_NUMBER}'
                            }
                        }
                    }
                }
            }
        }
    }
}
