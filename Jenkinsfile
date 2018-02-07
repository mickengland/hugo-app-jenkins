podTemplate(label: 'pod-hugo-app', containers: [
    containerTemplate(name: 'hugo', image: 'smesch/hugo', ttyEnabled: true, command: 'cat'),
    containerTemplate(name: 'html-proofer', image: 'smesch/html-proofer', ttyEnabled: true, command: 'cat'),
    containerTemplate(name: 'kubectl', image: 'smesch/kubectl', ttyEnabled: true, command: 'cat',
        volumes: [secretVolume(secretName: 'kube-config', mountPath: '/root/.kube')]),
    containerTemplate(name: 'docker', image: 'docker', ttyEnabled: true, command: 'cat',
        volumes: [
                  hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock')

  ]) {
    node('pod-hugo-app') {

        def DOCKER_HUB_ACCOUNT = 'mickengland'
        def DOCKER_IMAGE_NAME = 'hugo-app-jenkins'
        def K8S_DEPLOYMENT_NAME = 'hugo-app'


 stage('do some Docker work') {
            container('docker') {

                withCredentials([[$class: 'UsernamePasswordMultiBinding',
                        credentialsId: 'MEDockerHub',
                        usernameVariable: 'DOCKER_HUB_USER',
                        passwordVariable: 'DOCKER_HUB_PASSWORD']]) {

                    sh """
                        docker pull ubuntu
                        docker tag ubuntu ${env.DOCKER_HUB_USER}/ubuntu:${env.BUILD_NUMBER}
                        """
                    sh "docker login -u ${env.DOCKER_HUB_USER} -p ${env.DOCKER_HUB_PASSWORD} "
                    sh "docker push ${env.DOCKER_HUB_USER}/ubuntu:${env.BUILD_NUMBER} "
                }
            }
        }



        stage('Clone Hug App Repository') {
            checkout scm

            containter('hugo')
               stage{'Build Hugo Site') {
                    sh ("hugo --uglyURLs") 
               }
            }

            container('html-proofer') {
                stage('Validate HTML') {
                    sh ("htmlproofer public --internal-domains ${env.JOB_NAME} --external_only --only-4xx")
                }
            }

            container('docker') {

                withCredentials([[$class: 'UsernamePasswordMultiBinding',
                        credentialsId: 'MEDockerHub',
                        usernameVariable: 'DOCKER_HUB_USER',
                        passwordVariable: 'DOCKER_HUB_PASSWORD']]) {
                stage('Docker Build & Push Current & Latest Versions') {
                    sh ("docker build -t ${env.DOCKER_HUB_USER}/${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER} .")
                    sh ("docker tag ${env.DOCKER_HUB_USER}/${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER} ${DOCKER_HUB_ACCOUNT}/${DOCKER_IMAGE_NAME}:latest")
                    sh "docker login -u ${env.DOCKER_HUB_USER} -p ${env.DOCKER_HUB_PASSWORD} "
                    sh ("docker push ${env.DOCKER_HUB_USER}/${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}")
                    sh ("docker tag ${env.DOCKER_HUB_USER}/${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER} ${DOCKER_HUB_ACCOUNT}/${DOCKER_IMAGE_NAME}:latest")
                    sh ("docker push ${env.DOCKER_HUB_USER}/${DOCKER_IMAGE_NAME}:latest")
                }
            }

            container('kubectl') {

                withCredentials([[$class: 'UsernamePasswordMultiBinding',
                        credentialsId: 'MEDockerHub',
                        usernameVariable: 'DOCKER_HUB_USER',
                        passwordVariable: 'DOCKER_HUB_PASSWORD']]) {
                stage('Deploy New Build To Kubernetes') {
                    sh ("kubectl set image deployment/${K8S_DEPLOYMENT_NAME} ${K8S_DEPLOYMENT_NAME}=${DOCKER_HUB_ACCOUNT}/${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}")
             }
          }
        }
      }
    }
  }
}

