pipeline {
    agent any

    parameters {
        booleanParam(name: 'DEPLOY', defaultValue: false, description: 'Set to true to deploy the application')
    }

    stages {
        stage('SCM') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[url: 'https://github.com/Abhi-chintu/Springboot-project.git']]
                ])
            }
        }
        stage('Build artifacts') {
            steps {
                sh 'mvn clean install'
            }
        }
        stage('Docker image') {
            when {
                expression {
                    return params.DEPLOY
                }
            }
            steps {
                sh 'docker build -t abhishek508/spring-image:${BUILD_NUMBER} .'
            }
        }
        stage('Docker push') {
            when {
                expression {
                    return params.DEPLOY
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-cerds', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USER')]) {
                    sh '''
                    docker login -u ${DOCKER_USER} -p ${DOCKER_PASSWORD}
                    docker push abhishek508/spring-image:${BUILD_NUMBER}
                    '''
                }
            }
        }
        stage('Update K8s Manifests') {
            when {
                expression {
                    return params.DEPLOY
                }
            }
            steps {
                sh 'sed -i "s|image: abhishek508/spring-image:.*|image: abhishek508/spring-image:${BUILD_NUMBER}|g" k8s/deployment.yaml'
            }
        }
        stage('Deploy App on k8s') {
            when {
                expression {
                    return params.DEPLOY
                }
            }
            steps {
                dir('k8s/springboot-demo') {
                    withCredentials([string(credentialsId: 'my_kubernetes', variable: 'api_token')]) {
                        sh '''
                        kubectl config set-cluster my-cluster --server=https://172.31.4.92:8443 --insecure-skip-tls-verify=true
                        kubectl config set-credentials my-user --token=$api_token
                        kubectl config set-context my-context --cluster=my-cluster --user=my-user
                        kubectl config use-context my-context

                        # Deploy the Helm chart from the local directory with updated image tag
                        helm upgrade --install springboot-demo . --values values.yaml --set image.tag=${BUILD_NUMBER}
                        '''
                    }
                }
            }
        }
    }
    
    post {
        success {
            emailext(
                subject: "Jenkins Pipeline Success - Build #${BUILD_NUMBER}",
                body: "The Jenkins pipeline for Spring Boot project was successful. Build #${BUILD_NUMBER} has been deployed.",
                to: "abhi.n.j55@gmail.com"
            )
        }
    }
}
