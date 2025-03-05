pipeline {
    agent any

    environment {
        SONAR_URL = 'http://34.27.111.131:9000'
        SONAR_TOKEN = credentials('sonar-token')  // Add this in Jenkins credentials
        IMAGE_NAME = "saicharan12121/javasample1"
        BUILD_NUMBER = "${env.BUILD_NUMBER}"
        GOOGLE_APPLICATION_CREDENTIALS = credentials('gcp.json')
   
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
        sh '''
        docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .
        docker tag ${IMAGE_NAME}:${BUILD_NUMBER} ${IMAGE_NAME}:${BUILD_NUMBER}
        docker tag ${IMAGE_NAME}:${BUILD_NUMBER} ${IMAGE_NAME}:latest
        docker images

        '''
    }
}
        stage('Docker Login & Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh '''
                    echo $PASSWORD | docker login -u $USERNAME --password-stdin
                    docker push ${IMAGE_NAME}:${BUILD_NUMBER} || echo "Push Failed, Retrying..."
                    docker push ${IMAGE_NAME}:latest
                    '''
                }
            }
        }



        stage('Terraform Workspace Setup') {
    steps {
        sh '''
        cd terraform || exit 1
        rm -rf .terraform .terraform.lock.hcl
        terraform init -upgrade
        terraform plan
        terraform apply -auto-approve
        '''
    }
}

        //  stage('Terraform Apply') {
        //     steps {
           
        //          sh 'terraform init '
        //          sh 'terraform plan'
        //         sh 'terraform apply -auto-approve'
        
        //        // sh 'terraform destroy -auto-approve' //
        //     }
        // }
          
        
        stage("Check Connection") {
            steps {
                sh '''
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
