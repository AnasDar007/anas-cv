pipeline {
    agent any

    stages {
      stage('Build') {
        steps {
          script {
            dockerImage = docker.build("anasdaroo7/anas-cv:${env.BUILD_ID}")
        }
    }
}
        stage('Push') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Test') {
            steps {
                sh 'ls -l index.html' // Simple check for index.html
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // Deploy the new version
                    sshPublisher(
                        publishers: [
                            sshPublisherDesc(
                                configName: "AWS Assignment", 
                                transfers: [sshTransfer(
                                    execCommand: """
                                        docker pull anasdaroo7/anas-cv:${env.BUILD_ID}
                                        docker stop anas-cv-container || true
                                        docker rm anas-cv-container || true
                                        docker run -d --name anas-cv-container -p 80:80 anasdaroo7/anas-cv:${env.BUILD_ID}
                                    """
                                )]
                            )
                        ]
                    )

                    // Check if deployment is successful
                    boolean isDeploymentSuccessful = sh(script: 'curl -s -o /dev/null -w "%{http_code}" http://3.24.240.158:80', returnStdout: true).trim() == '200'

                    if (!isDeploymentSuccessful) {
                        // Rollback to the previous version
                        def previousSuccessfulTag = readFile('previous_successful_tag.txt').trim()
                        sshPublisher(
                            publishers: [
                                sshPublisherDesc(
                                    configName: "AWS Assignment",
                                    transfers: [sshTransfer(
                                        execCommand: """
                                            docker pull anasdaroo7/anas-cv:${previousSuccessfulTag}
                                            docker stop anas-cv-container || true
                                            docker rm anas-cv-container || true
                                            docker run -d --name anas-cv-container -p 80:80 anasdaroo7/anas-cv:${previousSuccessfulTag}
                                        """
                                    )]
                                )
                            ]
                        )
                    } else {
                        // Update the last successful tag
                        writeFile file: 'previous_successful_tag.txt', text: "${env.BUILD_ID}"
                    }
                }
            }
        }
    }

    post {
        failure {
            mail(
                to: 'm.anasdar336@gmail.com',
                subject: "Failed Pipeline: ${env.JOB_NAME} [${env.BUILD_NUMBER}]",
                body: "Something is wrong with the build ${env.BUILD_URL}"
            )
        }
    }
}
