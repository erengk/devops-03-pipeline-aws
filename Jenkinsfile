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
            DOCKER_LOGIN = "id_dockerhub_rwd"
            IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"
            IMAGE_TAG = "${RELEASE}.${BUILD_NUMBER}"
            JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")
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
                sh """
                  set -e

                  # 1) <none> (dangling) imajları temizle
                  docker image prune -f

                  # 2) Bu repo için en yeni 3 imajı KORU, kalanları sil (liste zaten oluşturulma tarihine göre sıralıdır)
                  docker images "${env.IMAGE_NAME}" --format '{{.ID}} {{.Repository}}:{{.Tag}}' \
                    | awk 'NR>3 {print \$1}' \
                    | sort -u \
                    | xargs -r docker rmi -f
                """
              } else {
                bat """
                  REM 1) <none> (dangling) imajları temizle
                  docker image prune -f

                  REM 2) Bu repo için en yeni 3 imajı KORU, kalanları sil
                  for /f "skip=3 tokens=1" %%i in ('docker images ${env.IMAGE_NAME} --format "{{.ID}}"') do docker rmi -f %%i
                """
              }
            }
          }
        }


        stage("Trigger CD Pipeline") {
            steps {
                script {
                    sh "curl -v -k --user erengk:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'ec2-18-204-175-140.compute-1.amazonaws.com:8080/job/devops-03-pipeline-aws-gitops/buildWithParameters?token=GITOPS_TRIGGER_START'"
                }
            }
        }

    }
}

        /*

         stage('Docker Image') {
             steps {
             //    sh 'docker build  -t erengk/devops-application:latest   .'
                 bat 'docker build  -t erengk/devops-application:latest   .'

             //   erengk/devops-03-pipeline-aws:latest
             //     ${DOCKER_USER}/${APP_NAME}:${IMAGE_TAG}
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