pipeline {
    agent {
        node {
            label 'MyDockerPC'
        }
    }
    //agent any
    tools {
        maven 'Maven3'
        jdk 'Java21'
    }

        environment {
            APP_NAME = "devops-03-pipeline-aws"
            RELEASE = "1.0"
            DOCKER_USER = "erengk"
            DOCKER_LOGIN = 'docker_jenkins_token'
            IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"
            IMAGE_TAG = "${RELEASE}.${BUILD_NUMBER}"
        }

    stages {
        stage('SCM GitHub') {
            steps {
                checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/erengk/devops-03-pipeline-aws']])
            }
        }
        stage('Test Maven') {
            steps {
                script {
                    if (isUnix()) {
                        // Linux or MacOS
                        sh 'mvn test'
                    } else {
                        bat 'mvn test'  // Windows
                    }
                }
            }
        }
        stage('Build Maven') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'mvn clean install'
                    } else {
                        bat 'mvn clean install'
                    }
                }
            }
        }

        stage("SonarQube Analysis") {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'Jenkins-SonarQube') {
                        if (isUnix()) {
                            // Linux or MacOS
                            sh "mvn sonar:sonar"
                        } else {
                            bat 'mvn sonar:sonar'  // Windows
                        }
                    }
                }
            }
        }

            stage("Quality Gate"){
                steps {
                     script {
                               waitForQualityGate abortPipeline: false, credentialsId: 'Jenkins-SonarQube'
                            }
                      }
                }
            }
        }

        stage('Build & Push Docker Image to DockerHub') {
             steps {
                 script {

                      docker.withRegistry('', DOCKER_LOGIN) {

                              docker_image = docker.build "${IMAGE_NAME}"
                              docker_image.push("${IMAGE_TAG}")
                              docker_image.push("latest")
                            }
                        }
                    }
                }

        stage("Trivy Scan") {
            steps {
                script {
                    if (isUnix()) {
                        sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image erengk/devops-03-pipeline-aws:latest --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table')
                     } else {
                        bat ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image erengk/devops-03-pipeline-aws:latest --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table')
                    }
                }
            }
        }




        stage('Cleanup Old Docker Images') {
            steps {
                script {
                    if (isUnix()) {
                        // Bu repo için tüm image’leri al, tarihe göre sırala, son 3 hariç sil
                        sh """
                            docker images "${env.IMAGE_NAME}" --format "{{.Repository}}:{{.Tag}} {{.CreatedAt}}" \\
                            | sort -r -k2 \\
                            | tail -n +4 \\
                            | awk '{print \$1}' \\
                            | xargs -r docker rmi -f
                        """

                    } else {
                        bat """
                             for /f "skip=3 tokens=1" %%i in ('docker images ${env.IMAGE_NAME} --format "{{.Repository}}:{{.Tag}}" ^| sort') do docker rmi -f %%i
                        """
                    }
                }
            }
        }




        /*

         stage('Docker Image') {
             steps {
             //    sh 'docker build  -t erengk/devops-application:latest   .'
                 bat 'docker build  -t erengk/devops-application:latest   .'
             }
         }


         stage('Docker Image To DockerHub') {
             steps {
                 script {
                     withCredentials([string(credentialsId: 'dockerhub', variable: 'dockerhub')]) {

                         if (isUnix()) {
                              sh 'docker login -u erengk -p %dockerhub%'
                              sh 'docker push erengk/devops-application:latest'
                           } else {
                              bat 'docker login -u erengk -p %dockerhub%'
                              bat 'docker push erengk/devops-application:latest'
                          }
                     }
                 }
             }
         }


         stage('Deploy Kubernetes') {
             steps {
             script {
                     kubernetesDeploy (configs: 'deployment-service.yaml', kubeconfigId: 'kubernetes')
                 }
             }
         }


         stage('Docker Image to Clean') {
             steps {

                    //  sh 'docker image prune -f'
                      bat 'docker image prune -f'

             }
         }

 */