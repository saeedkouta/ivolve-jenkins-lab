# Description
## Set up pipeline to deploy App. Using Jenkinsfile
### Steps:
#### *Build Docker image from Dockerfile in GitHub
#### *Push image to Docker hub
#### *Edit new image in deployment.yaml file
#### *Deploy to OC
#### *Use Jenkins shared libraries
#### *Set pipeline post action (always, success, failure)

### Step 1: Create Deployment and Service Files

#### Deploymentfile With image that will be updated:

<img src="https://github.com/saeedkouta/ivolve-jenkins-lab2/assets/167209058/865fe490-957f-4a0f-977a-d934b3a87776" width="1000" > 

#### Servicefile:

<img src="https://github.com/saeedkouta/ivolve-jenkins-lab2/assets/167209058/a64bdc9e-01a1-4765-a788-0a351d1c2927" width="1000" > 

### Step 1: Create Jenkinsfile

#### *using Shared Library:
```groovy
@Library('shared-library')_
```

#### *using Agent Any:
```groovy
agent any
```

#### *using 4 Stages (Build, Push, Deploy, Update)
```groovy
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
```

#### *using Post (always, success, failure)
```json
    post {
        always {
            echo "Pipeline completed."
        cleanWs() // Clean workspace after the build is finished
        }
        success {
            echo "${JOB_NAME}-${BUILD_NUMBER} pipeline succeeded"
        }
        failure {
            echo "${JOB_NAME}-${BUILD_NUMBER} pipeline failed"
        }
    }
```

### Stage 2: Create Shared Library

#### *buildDockerImage.groovy
```groovy
#!usr/bin/env groovy
def call(String imageName) {

        // Build and push Docker image
        echo "Building Docker image..."
        sh "docker build -t ${imageName}:v1 ."
}
```

#### *pushDockerImage.groovy
```groovy
#!usr/bin/env groovy
def call(String dockerHubCredentialsID, String imageName) {

	// Log in to DockerHub 
	withCredentials([usernamePassword(credentialsId: "${dockerHubCredentialsID}", usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
		sh "docker login -u ${USERNAME} -p ${PASSWORD}"
        }
        
        // Build and push Docker image
        echo "Pushing Docker image..."
        sh "docker push ${imageName}:v1"	 
}
```

#### *deployToOpenShift.groovy
```groovy
#!/usr/bin/env groovy

//OpenShiftCredentialsID can be credentials of service account token or KubeConfig file 
def call(String OpenShiftCredentialsID, String openshiftClusterurl, String openshiftProject, String imageName) {
    
    // login to OpenShift Cluster via cluster url & service account token
    withCredentials([string(credentialsId: "${OpenShiftCredentialsID}", variable: 'OpenShift_CREDENTIALS')]) {
            sh "oc login --server=${openshiftClusterurl} --token=${OpenShift_CREDENTIALS} --insecure-skip-tls-verify"
            sh "oc apply -f deployment.yml"
            sh "oc apply -f service.yml"
    }
}
```

[This is the repo of the Shared Library](https://github.com/saeedkouta/jenkins-oc-shared-library.git)

#### updateDeployment.groovy
```groovy
def call(String imageName, BUILD_NUMBER) {
    // Update the image in the deployment.yml file
    sh """
    sed -i 's|image: .*|image: ${imageName}:v1|g' deployment.yml
    """
}
```

### Step 3: Create Jenkins-Matser

#### Iam Using As docker Container As Jenkins-Master and this is it's Configration
```bash
sudo docker run -p 8080:8080 -p 50000:50000 -d \
 -v jenkins_home:/var/jenkins_home \
 -v /var/run/docker.sock:/var/run/docker.sock \
 -v $(which docker):/usr/bin/docker jenkins/jenkins 
```

### Step 4: Configure the shared library on the jenkins system

<img src="https://github.com/saeedkouta/Jenkins-OC-Project/assets/167209058/f29eec84-3913-4ceb-b89c-101d3145bf6a" width="1000" > 

<img src="https://github.com/saeedkouta/Jenkins-OC-Project/assets/167209058/16af7cea-c0e3-4a47-84f3-079de3a9168f" width="1000" > 

### Step 5: Add Credentials  

#### *OC-Token:

<img src="https://github.com/saeedkouta/Jenkins-OC-Project/assets/167209058/9f2f5cbe-c92e-43da-bba4-541736828114" width="1000" > 

#### *GitHub:

<img src="https://github.com/saeedkouta/Jenkins-OC-Project/assets/167209058/4115c0f2-3f66-423b-876b-e673b16623e5" width="1000" > 

#### *DockerHub

<img src="https://github.com/saeedkouta/Jenkins-OC-Project/assets/167209058/a7539510-6676-4a49-af22-e948cde9918e" width="1000" > 

### Step 6: Create The Pipeline 

#### 1- Create a Pipeline and add the repo of the project on it:

<img src="https://github.com/saeedkouta/ivolve-jenkins-lab2/assets/167209058/aa7e6056-f9f2-4d6c-9203-832d3aab4a38" width="1000" > 

<img src="https://github.com/saeedkouta/ivolve-jenkins-lab2/assets/167209058/31afffba-ef2a-468f-b233-9961301117a1" width="1000" > 

<img src="https://github.com/saeedkouta/ivolve-jenkins-lab2/assets/167209058/588e35a6-507f-4f9c-a2bf-9f642766a254" width="1000" > 

#### 2- Build The Pipeline:

<img src="https://github.com/saeedkouta/ivolve-jenkins-lab2/assets/167209058/8ffae792-d3e2-44b3-ba29-595d8c7d2e76" width="1000" > 

#### 3- Ensure That the image pushed to DockerHub

<img src="https://github.com/saeedkouta/ivolve-jenkins-lab2/assets/167209058/147dc6a5-0a1b-403b-8996-cbe0195f54cc" width="1000" > 

### Step 11: Ensure That the Deploy is Created And use port-forward to see The Website

<img src="https://github.com/saeedkouta/ivolve-jenkins-lab2/assets/167209058/ff6406d8-1003-4461-b028-14e04b37289c" width="1000" > 

<img src="https://github.com/saeedkouta/ivolve-jenkins-lab2/assets/167209058/d9717519-770e-4aa9-b799-04c0165b26ce" width="1000" > 

<img src="https://github.com/saeedkouta/ivolve-jenkins-lab2/assets/167209058/dbaafd9b-e643-4471-9393-abe388b908e9" width="1000" > 

