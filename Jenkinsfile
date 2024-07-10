pipeline{
	agent any 
	tools{
		jdk 'jdk17'
		nodejs 'node16'
	}
	environment{
		SCANNER_HOME=tool 'sonar-scanner'
	}
	stages{
		stage('clean workspace'){
			steps{
				cleanWs()
			}
		}
		stage('Code Checkout From Git'){
		   steps{
			git branch:'main', url: 'https://github.com/neelesh9346/Zomato-Project.git'
		   }
		}
		stage("SonarQube Code Analysis"){
		    steps{
			withSonarQubeEnv('sonar-scanner'){
			  sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=zomato \
			  -Dsonar.projectKey=zomato '''
			}
		    }
		}
		stage("Code Quality Gates"){
		    steps{
			script{
			    timeout(time: 2, unit:'MINUTES'){
			    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
			}
		    }
		}
	    }
	        stage("Install Dependencies"){
		    steps{
			sh "npm install"
		    }
		}
		stage("OWASP FS SCAN"){
		  steps{
		      dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
		      dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
		  }
		}
		stage("TRIVY FS SCAN"){
		  steps{
		     sh "trivy fs . > trivy.txt"
		  }
		}
		stage("DOcker Image Build and Push"){
		    steps{
			script{
			withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
			    sh "docker build -t cloudzomato . "
			    sh "docker tag cloudzomato neelesh9346/cloudzomato:latest"
			    sh "docker push neelesh9346/cloudzomato:latest"
			}
			}
		    }
		}
	stage("TRIVY is Image Scanning"){
	    steps{
		sh "trivy image neelesh9346/cloudzomato:latest >trivy.txt"
	    }
	}
	stage("Creating Docker Container "){
	    steps{
		sh "docker run -d --name zomato-app ph zomato -p 3000:3000 neelesh9346/cloudzomato:latest"
	    }
	}



	}
}
