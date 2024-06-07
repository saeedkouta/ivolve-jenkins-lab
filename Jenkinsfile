@Library('shared-library')_
pipeline {
    agent any
    
    environment {
        dockerHubCredentialsID = 'DockerHub'                          // DockerHub credentials ID.
        imageName              = 'saeedkouta/jenkins-lab2'         // DockerHub repo/image name.
        openshiftCredentialsID = 'openshift-token'                    // service account token credentials ID
        openshiftClusterURL    = 'https://api.ocp-training.ivolve-test.com:6443' // OpenShift Cluster URL.
        openshiftProject       = 'saeedkouta'                         // OpenShift project name.
    }

    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    // Navigate to the directory that contains Dockerfile
                    buildDockerImage("${imageName}")
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    // Navigate to the directory that contains Dockerfile
                    pushDockerImage("${dockerHubCredentialsID}", "${imageName}")
                }
            }
        }

        stage('Deploy on OpenShift Cluster') {
            steps {
                script { 
                    deployToOpenShift("${openshiftCredentialsID}", "${openshiftClusterURL}", "${openshiftProject}", "${imageName}")
                }
            }
        }
        stage('Update Deployment YAML') {
            steps {
                script {
                    updateDeployment("${imageName}", "${BUILD_NUMBER}")
                }
            }
        }
    }

    post {
        success {
            echo "${JOB_NAME}-${BUILD_NUMBER} pipeline succeeded"
        }
        failure {
            echo "${JOB_NAME}-${BUILD_NUMBER} pipeline failed"
        }
    }
}

