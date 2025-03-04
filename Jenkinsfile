pipeline {
    agent any
    environment {
        SONAR_URL = 'http://34.27.111.131:9000'
        SONAR_TOKEN = credentials('sonar-token')  // Add this in Jenkins credentials
        IMAGE_NAME = "saicharan12121/myappjekinspush"
   
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/Saicharan619/mvnsonardoc.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {  // This automatically sets SONAR_URL and SONAR_TOKEN
                sh '''
                mvn sonar:sonar \
                    -Dsonar.projectKey=my-project \
                    -Dsonar.token=${SONAR_TOKEN}
                '''
               }
            }
        }
       stage("Quality Gate") {
            steps {
              timeout(time: 10, unit: 'MINUTES') {
                waitForQualityGate abortPipeline: true
              }
            }
          }
        

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${IMAGE_NAME} .'
            }
        }

        stage('Docker Login & Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh '''
                    echo $PASSWORD | docker login -u $USERNAME --password-stdin
                    docker push ${IMAGE_NAME}:${BUILD_NUmber}
                    '''
                }
            }
        }
        stage("Check Connection") {
            steps {
                sh '''
                cd /etc/ansible/dynamic/
                ansible-inventory --graph
                '''
            }
        }

        stage("Ping Ansible") {
            steps {
                sh '''
                sleep 10
                ansible all -m ping
                '''
            }
        }

         stage('Ansible Deployment') {
            steps {
                sh '''
                ansible-playbook dockerinstall.yml -e build_number=$BUILD_NUMBER
                '''
            }
        }


        stage('Run Docker Container') {
            steps {
                sh 'docker run -d -p 8085:8081 --name myapp_container ${IMAGE_NAME}:${BUILD_NUMBER}'
            }
        }

       
    }
        

    post {
        success {
            echo "Application successfully deployed in a Docker container!"
        }
        failure {
            echo "Build failed!"
        }
    }
}
