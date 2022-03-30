pipeline {
    agent none
    tools{
        jdk 'myJava'
        maven 'myMaven'
    }
    stages {
        stage('COMPILE') {
            agent any
            steps {
               script {
                   echo "COMPILING THE CODE"
                   git 'https://github.com/vcvishalchand/addressbook.git'
                   sh 'mvn compile'
                }
            }
        }
        stage('UNITTEST') {
            //agent {label 'linux_slave'}
            agent any
            steps {
               script {
                   echo "RUNNING THE UNIT TEST CASES"
                   sh 'mvn test'
                }
            }
            post{
                always{
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        stage('PACKAGE+BUILD DOCKER IMAGE ON BUILD SERVER'){     
            agent any       
            steps {
                script {
                    sshagent(['Test_server-Key']) {
                        withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                            echo "PACKAGING THE CODE"
                            sh "scp -o StrictHostKeyChecking=no server-script.sh ec2-user@172.31.14.250:/home/ec2-user"
                            sh "ssh -o StrictHostKeyChecking=no ec2-user@172.31.14.250 'bash ~/server-script.sh'"
                            sh "ssh ec2-user@172.31.14.250 sudo docker build -t vishalchand/java-mvn-repos:$BUILD_NUMBER /home/ec2-user/addressbook"
                            sh "ssh ec2-user@172.31.14.250 sudo docker login -u $USERNAME -p $PASSWORD"
                            sh "ssh ec2-user@172.31.14.250 sudo docker push vishalchand/java-mvn-repos:$BUILD_NUMBER"
                        }
                    }
                }
            }
        }

        stage('Deploy the docker image') {
            agent any
                steps {
                    script {
                        sshagent(['Test_server-Key']) {
                            withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                                sh "ssh ec2-user@172.31.33.78 sudo docker run -itd -P vishalchand/java-mvn-repos:$BUILD_NUMBER"
                            }
                        }
                    }
                }            
        }

    }
}
