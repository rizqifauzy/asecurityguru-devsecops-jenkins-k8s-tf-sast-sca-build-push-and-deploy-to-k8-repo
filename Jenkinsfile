pipeline {
  agent any
  tools { 
        maven 'maven-3.5.2'  
    }
   stages{
    stage('CompileandRunSonarAnalysis') {
            steps {	
		sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=arf-attck-webapp -Dsonar.organization=arf-attck -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=67c391f24f7f77947fe60f984072bdb60dbf731b'
			}
    }

	stage('RunSCAAnalysisUsingSnyk') {
            steps {		
				withCredentials([string(credentialsId: 'SNYK_TOKEN', variable: 'SNYK_TOKEN')]) {
					sh 'mvn snyk:test -fn'
				}
			}
    }

	stage('Build') { 
            steps { 
               withDockerRegistry([credentialsId: "dockerlogin", url: ""]) {
                 script{
                 app =  docker.build("arf-devsecops")
                 }
               }
            }
    }

	stage('Push') {
            steps {
                script{
                    docker.withRegistry('https://199318128185.dkr.ecr.ap-southeast-3.amazonaws.com', 'ecr:ap-southeast-3:aws-credentials') {
                    app.push("latest")
                    }
                }
            }
    	}
	   
	stage('Kubernetes Deployment of ASG Bugg Web Application') {
	   steps {
	      withKubeConfig([credentialsId: 'kubelogin']) {
		  sh('kubectl delete all --all -n devsecops')
		  sh('kubectl apply -f deployment.yaml --namespace=devsecops')
		}
	      }
   	}

  }
}
