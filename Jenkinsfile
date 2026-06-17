pipeline {

    agent any

    tools {
        maven 'mvn_3.9.16' // This should match the Maven name in Jenkins Global Tool Configuration
    }

    stages{
    
    stage('Checkout from GitHub') {
        steps {
            git branch: 'feature',
                url: 'https://github.com/KandlaguntaVenkataSivaNiranjanReddy/spring-boot-mongo-docker-kkfunda.git'
        }
      }

    stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

     stage('SonarQube') {
            steps {

                withSonarQubeEnv('sonarQube') {

                    sh '''
                    mvn sonar:sonar \
                    -Dsonar.projectKey=spring-boot-mongo \
                    -Dsonar.projectName="Spring Boot Mongo Project"
                    '''

                }
            }
        }
      
      stage('Build Docker Image') {
            steps {

                script {

                    withDockerRegistry(credentialsId: 'docker') {

                        sh 'docker build -t satyamolleti4599/mongospring:1.0.0 .'

                    }
                }
            }
        }

      stage('Push Docker Image') {
            steps {

                script {

                    withDockerRegistry(credentialsId: 'docker') {

                        sh 'docker push satyamolleti4599/mongospring:1.0.0'

                    }
                }
            }
        }

      stage('Setup KubeConfig') {
        steps {
            withCredentials([
                aws(
                    accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                    credentialsId: 'aws-eks-cred',
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                )
            ]) {
                sh '''
                    aws eks update-kubeconfig --region ap-south-2 --name EKS-CLUSTER
                '''
            }
        }
    }

    stage('Deploy to Kubernetes') {
    steps {
        withCredentials([
            aws(
                accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                credentialsId: 'aws-eks-cred',
                secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
            )
        ]) {
            sh '''
                kubectl apply -f springappmongo.yaml --validate=false
            '''
          }
        }
    }

    stage('Verify Pods and Services') {
        steps {
            withCredentials([
                aws(
                    accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                    credentialsId: 'aws-eks-cred',
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                )
            ]) {
                sh '''
                    kubectl get pods
                    kubectl get svc
                '''
            }
        }
    }
     
    }

}
