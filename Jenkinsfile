node{
    
    def docker, tag, dockerHubUser, backendContainerName, frontendContainerName,
    backendHttpPort, frontendHttpPort = ""
     stage('Prepare Environment'){
       echo 'Initialize Environment'
       tag="latest"
                 withCredentials([usernamePassword(credentialsId: 'dockerHubAccount',

    usernameVariable: 'dockerUser', passwordVariable: 'dockerPassword')]) {

                       dockerHubUser="$dockerUser"

    }
                backendContainerName="insure-me-backend"
                frontendContainerName="insure-me-frontend"
                backendHttpPort="8080"
                frontendHttpPort="3000"
    }
    
    stage('Code Checkout'){
        try{
            checkout scm
        }
        catch(Exception e){
            echo 'Exception occured in Git Code Checkout Stage'
            currentBuild.result = "FAILURE"
            //emailext body: '''Dear All,
            //The Jenkins job ${JOB_NAME} has been failed. Request you to please have a look at it immediately by clicking on the below link. 
            //${BUILD_URL}''', subject: 'Job ${JOB_NAME} ${BUILD_NUMBER} is failed', to: 'jenkins@gmail.com'
        }
    }
    
    stage('Backend Maven Build'){
         sh "mvn clean package"        
    }
    stage('FrontEnd NodeJS Build'){
        dir("frontend"){
                   sh """
                           npm install
                           npm run test
                           npm run build
                     """
	}
    } 
    
    stage('Backend Docker Image Build'){
        echo 'Creating Docker image'
        sh "docker build -t $dockerHubUser/$backendContainerName:$tag --pull --no-cache ."
      }
      stage('Frontend Docker Image Build'){
       dir("frontend"){

                        echo 'Creating Docker image'
                        sh "docker build -t $dockerHubUser/$frontendContainerName:$tag --pull --no-cache ."

    
                   }

      }
    stage('Docker Image Scan'){
        echo 'Scanning Docker image for vulnerbilities'
        sh "docker build -t ${dockerHubUser}/insure-me:${tag} ."
    }   
	
    stage('Publishing Image to DockerHub'){
        echo 'Pushing the docker image to DockerHub'
        withCredentials([usernamePassword(credentialsId: 'dockerHubAccount', usernameVariable: 'dockerUser', passwordVariable: 'dockerPassword')]) {
			sh "docker login -u $dockerUser -p $dockerPassword"
			sh "docker push $dockerUser/$backendContainerName:$tag"
		        sh "docker push $dockerUser/$frontendContainerName:$tag"
			echo "Image push complete"
        } 
    }    
	
	stage('Docker Container Deployment'){
		node('DockerHost'){
		sh "docker rm $backendContainerName -f"
		sh "docker rm $frontendContainerName -f"
		sh "docker run -d --rm -p $backendHttpPort:$backendHttpPort --name $backendContainerName $dockerHubUser/$backendContainerName:$tag"
		sh "docker run -d --rm -p $frontendHttpPort:$frontendHttpPort --name $frontendContainerName $dockerHubUser/$frontendContainerName:$tag"
		echo "Application started on port: ${frontendHttpPort} (http)"
	}
   }
 } 

