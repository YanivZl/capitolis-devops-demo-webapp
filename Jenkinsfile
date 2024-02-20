podTemplate(label: 'builder',
            containers: [
                    containerTemplate(name: 'nodejs', image: 'node:21-alpine3.18', command: 'cat', ttyEnabled: true, privileged: true),
                    containerTemplate(name: 'dind', image: 'odavid/jenkins-jnlp-slave:4.10-2-31-jdk11', command: '/usr/local/bin/wrapdocker', ttyEnabled: true, privileged: true),
                    containerTemplate(name: 'helm-k8s', image: 'lachlanevenson/k8s-helm:v3.10.2', command: 'cat', ttyEnabled: true, privileged: true)
            ]) {
        node('builder') {
            parameters {
                choice (
                    name: 'CD Tool',
                    choices: 'Helm\nArgoCD',
                    description: 'Choose deploy tool'
                )
            }
            
            DOCKER_IMAGE_NAME = env.JOB_NAME.takeWhile { it != '/' }

            stage('Checkout') {
                checkout scm
            }

            stage('Build') {
                container('nodejs') {
                    sh '''
                    pwd
                    ls -l
                    # install npm deps
                    mkdir -p node_modules && chown -R node:node node_modules
                    chmod -R 775 /root
                    chown -R 1000:1000 "/root/.npm"
                    npm install
                    '''
                }
            }

            stage('Build Docker') {
                container('dind') {
                    script {
                        withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId:'docker-hub', usernameVariable: 'DOCKER_HUB_USERNAME', passwordVariable: 'DOCKER_HUB_PASSWORD']]) {
                            dockerImage = docker.build("${DOCKER_HUB_USERNAME}/${DOCKER_IMAGE_NAME}:${BUILD_NUMBER}")
                        }
                        sh "docker images"
                    }
                }
            }

            stage('Push to Registry') {
                container('dind') {
                    script {
                        withDockerRegistry([ credentialsId: "docker-hub", url: "" ]) {
                            dockerImage.push()
                        }
                    }
                }
            }

            stage('Trigger CD Pipeline') {
                echo "triggering updatemanifestjob"
                build job: "${DOCKER_IMAGE_NAME}-cd", parameters: [string(name: 'DOCKERTAG', value: env.BUILD_NUMBER)]
            }



        }
    }