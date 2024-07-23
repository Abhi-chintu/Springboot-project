pipeline {
    agent any

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
            steps {
                sh 'docker build -t abhishek508/spring-image:${BUILD_NUMBER} .'
            }
        }
        stage('Docker push'){
            steps {
            withCredentials([usernamePassword(credentialsId: 'docker-creds', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USER')]) {
                sh '''
                docker login -u ${DOCKER_USER} -p ${DOCKER_PASSWORD}
                docker push abhishek508/spring-image:${BUILD_NUMBER}
                '''
                }
            }
        }
        stage('Update K8s Manifests') {
            steps {
                sh 'sed -i "s|image: abhishek508/spring-image:.*|image: abhishek508/spring-image:${BUILD_NUMBER}|g" k8s/deployment.yaml'
            }
        }
        stage('Deploy App on k8s') {
      steps {
          dir('k8s') {
        withCredentials([
            string(credentialsId: 'my_kubernetes', variable: 'api_token')]) {
             sh 'kubectl --token $api_token --server https://172.31.2.119:8443  --insecure-skip-tls-verify=true apply -f deployment.yaml '
             sh 'kubectl --token $api_token --server https://172.31.2.119:8443  --insecure-skip-tls-verify=true apply -f service.yaml'
                  }
               }
            }
        }
    }
}
