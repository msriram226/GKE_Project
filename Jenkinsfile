pipeline 
{
    agent any
	tools {
		maven 'maven'
	}
	environment {
		BUILD_ID = getDockerTag()
        IMAGE_WITH_TAG = 'msriram226/gcp_devops_project:${BUILD_ID}'
		PROJECT_ID = 'axiomatic-folio-332019'
                CLUSTER_NAME = 'k8s-cluster'
                LOCATION = 'us-west4-b'
                CREDENTIALS_ID = 'kubernetes'		
	}
	
    stages {
	    stage('Scm Checkout') {
		    steps {
			    checkout scm
		    }
	    }
	    
	    stage('Build') {
		    steps {
			    sh 'mvn clean package'
		    }
	    }
	    
	    stage('Test') {
		    steps {
			    echo "Testing..."
			    sh 'mvn test'
		    }
	    }
	    
	    stage('Build Docker Image') {
		    steps {
			    sh 'whoami'
			    script {
				    sh "docker build . -t ${IMAGE_WITH_TAG}"
			    }
		    }
	    }
	    
	    stage("Push Docker Image") {
		    steps {
			    script {
				    echo "Push Docker Image"
				        withCredentials([string(credentialsId: 'dockerhub', variable: 'dockerhub')]) {
            			sh "docker login -u msriram226 -p ${dockerhub}" } {
                        sh "docker push ${IMAGE_URL_WITH_TAG}"  }
				    }  
			  
			    }
		    }
	    }
	    
	    stage('Deploy to K8s') {
		    steps{
			    echo "Deployment started ..."
			    sh 'ls -ltr'
			    sh 'pwd'
			    sh "sed -i 's/tagversion/${env.BUILD_ID}/g' serviceLB.yaml"
				sh "sed -i 's/tagversion/${env.BUILD_ID}/g' deployment.yaml"
			    echo "Start deployment of serviceLB.yaml"
			    step([$class: 'KubernetesEngineBuilder', projectId: env.PROJECT_ID, clusterName: env.CLUSTER_NAME, location: env.LOCATION, manifestPattern: 'serviceLB.yaml', credentialsId: env.CREDENTIALS_ID, verifyDeployments: true])
				echo "Start deployment of deployment.yaml"
				step([$class: 'KubernetesEngineBuilder', projectId: env.PROJECT_ID, clusterName: env.CLUSTER_NAME, location: env.LOCATION, manifestPattern: 'deployment.yaml', credentialsId: env.CREDENTIALS_ID, verifyDeployments: true])
			    echo "Deployment Finished ..."
		    }
        }
    }
}
def getDockerTag()
{
    def tag  = sh script: 'git rev-parse HEAD', returnStdout: true
    return tag
}
