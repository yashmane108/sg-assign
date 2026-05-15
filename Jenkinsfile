pipeline {
    agent any

    environment {
        BRANCH = "${env.BRANCH_NAME}"
        NAMESPACE = "pr-${env.BRANCH}"
        IMAGE = "yashmane108/simple-web:${BRANCH}-${BUILD_NUMBER}"
        // Manifest_file_path = "deployment.yaml"
    }

    stages {

        stage('Clone Repo') {
            steps {
                echo "Branch: ${env.BRANCH}"
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE} ." 
            }
        }

        stage("push to docker hub") {
            steps {
                withCredentials([usernamePassword(
                credentialsId:"dockerCred",
                passwordVariable: "dockerHubPass",
                usernameVariable: "dockerHubUser"
                )]){
                sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPass}"
                //sh "docker image tag simple-web:${env.BUILD_NUMBER} ${env.dockerHubUser}/simple-web:${env.BUILD_NUMBER}"
                sh "docker push ${env.dockerHubUser}/simple-web:${BRANCH}-${BUILD_NUMBER}"
                }    
             }
         }
        
        stage('Deployment Edit') {
            steps {
                sh "sed -i 's|REPLACE|${IMAGE}|g' deployment.yaml"
            }
        }
        
        stage('Create Namespace') {
            steps {
                sh "kubectl create namespace ${env.NAMESPACE} || true"
            }
        }

        stage('Deploy') {
            steps {
                sh """
                kubectl apply -f deployment.yaml -n ${env.NAMESPACE}
                kubectl apply -f service.yaml -n ${env.NAMESPACE}
                """
            }
        }

        // stage('Cleanup Old PR Namespaces') {
        //     when {
        //         branch 'staging'
        //     }

        //     steps {
        //         echo "Waiting 10 minutes before cleanup..."
        //         sleep time: 10, unit: 'MINUTES'
        //         sh """
        //         kubectl delete namespace pr-feature-login --ignore-not-found=true
        //         kubectl delete namespace pr-feature-search --ignore-not-found=true
        //         """
        //     }
        // }
    }
}
