def dockerRepository = "wattt3"
def dockerImageName = "study"
def deployHost = "13.125.220.216"

pipeline {
    agent any

    stages {
        stage('Checkout'){
            steps {
                checkout scm
            }
        }
        stage('Build') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-key',
                                 usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    script {
                        sh """
                            chmod u+x ./gradlew
                            ./gradlew clean build -Pdocker.repository=${dockerRepository} \
                                                  -Pdocker.repository.username=${USERNAME} \
                                                  -Pdocker.repository.password=${PASSWORD} \
                                                  -Pdocker.image.name=${dockerImageName} \
                                                  -Pdocker.image.tag=${currentBuild.number}
                        """
                    }
                }
            }
        }
        stage('Publish') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-key',
                                 usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    script {
                        sh """
                            ./gradlew jib -Pdocker.repository=${dockerRepository} \
                                          -Pdocker.repository.username=${USERNAME} \
                                          -Pdocker.repository.password=${PASSWORD} \
                                          -Pdocker.image.name=${dockerImageName} \
                                          -Pdocker.image.tag=${currentBuild.number}
                        """
                    }
                }
            }
        }
        stage('Deploy'){
            steps {
                sshagent(credentials: ["deploy-key"]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ec2-user@${deployHost} \
                        'docker container rm -f springbootapp &&
                         docker container run -d -t -p 80:8080 --rm --name springbootapp ${dockerRepository}/${dockerImageName}:${currentBuild.number};'
                    """
                }
            }
        }
    }
}