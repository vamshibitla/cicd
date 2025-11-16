pipeline {
    agent any


       tools {
        maven 'maven3'
    }

    stages {
      stage('checkout') {
            steps {
                echo 'Cloning GIT HUB Repo '
	git branch: 'master', url: 'https://github.com/vamshibitla/cicd.git'
            }  
        }
	
	
		
		
		
	stage('sonar') {
			steps {
				echo 'scanning project'
                sh 'ls -ltr'
				withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
				sh '''
				echo "Running Sonar scan..."
					mvn sonar:sonar \
					-Dsonar.host.url=http://52.87.44.103:9000 \
					-Dsonar.login=$SONAR_TOKEN
				'''
            }
        }
    }
	
		
        stage('Build Artifact') {
            steps {
                echo 'Build Artifact'
	sh 'mvn clean install'
            }
        }
	
	    stage('Nexus Deploy') {
			steps {
				echo "Deploying artifact to Nexus..."

				withCredentials([usernamePassword(
				credentialsId: 'nexus-cred',   // <-- your Jenkins credential ID
				usernameVariable: 'NEXUS_USER',
				passwordVariable: 'NEXUS_PASS'
				)]) {

				sh '''
                mvn clean deploy -DskipTests=true \
                  -DaltDeploymentRepository=nexus-releases::default::http://52.87.44.103:8081/repository/nexus-repo/ \
                  -Dusername=$NEXUS_USER -Dpassword=$NEXUS_PASS
				'''
        }
    }
}

	
        stage('Docker Image') {
            steps {
                echo 'Docker Image building'
	sh 'docker build -t vamsi01/javaproject:${BUILD_NUMBER} .'
            }
        }
	
	
       stage('Push to Dockerhub') {
    steps {
        script {
            withCredentials([
                usernamePassword(
                    credentialsId: 'dockerhub',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )
            ]) {
                sh '''
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    
                    # Build image
                    docker build -t $DOCKER_USER/javaproject:${BUILD_NUMBER} .
                    
                    # Push image
                    docker push $DOCKER_USER/javaproject:${BUILD_NUMBER}
                '''
            }
        }
    }
}
	
	
    stage('Update Deployment File') {
	
	 environment {
            GIT_REPO_NAME = "cicd"
            GIT_USER_NAME = "vamshibitla"
        }
	
            steps {
                echo 'Update Deployment File'
	withCredentials([string(credentialsId: 'githubtoken', variable: 'githubtoken')]) 
	{
                  sh '''
                    git config user.email "vamshi123.bitla@gmail.com"
                    git config user.name "vamshi"
                    BUILD_NUMBER=${BUILD_NUMBER}
                    sed -i "s/javaproject:.*/javaproject:${BUILD_NUMBER}/g" deploymentfiles/deployment.yml
                    git add .
                    
                    git commit -m "Update deployment image to version ${BUILD_NUMBER}"

                    git push https://${githubtoken}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                '''
	  
                 }
	
            }
        }
	
	
		
    }

}
