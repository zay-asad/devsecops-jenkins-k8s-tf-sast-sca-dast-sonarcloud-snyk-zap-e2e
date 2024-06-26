pipeline {
  agent any
  tools { 
        maven 'Maven_3_5_2'  
    }
   stages{
    stage('CompileandRunSonarAnalysis') {
            steps {	
		sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=zbuggywebapp -Dsonar.organization=zbuggywebapp -Dsonar.host.url=https://sonarcloud.io -Dsonar.token=245530707f179f57bb2c7ae6f89335ff5c328f3c'
			}
        }

   stage('RunSCAAAnalysisUsingSnyk') {
    steps {	
		withCredentials([string(credentialsId: 'SNYK_TOKEN', variable: 'SNYK_TOKEN')]) {
			sh 'mvn snyk:test -fn'
		}
	} 
  }

   stage('Build') {
    steps {	
      withDockerRegistry([credentialsId: "dockerlogin", url: ""]) {
        script {
          app = docker.build("zdevsecops")
        }
      }
		}
	} 
  
   stage('Push') {
    steps {	
      script {
        docker.withRegistry('https://730335353514.dkr.ecr.us-west-2.amazonaws.com', 'ecr:us-west-2:aws-credentials') {
          app.push("latest")
        }
      }
		}
	}
  
    stage('Kubernetes Deployment of ASG Bugg Web Application') {
      steps {
          withKubeConfig([credentialsId: 'kubelogin']) {
        sh('kubectl delete all --all -n devsecops')
        sh ('kubectl apply -f deployment.yaml --namespace=devsecops')
      }
          }
      }

    stage ('wait_for_testing'){
      steps {
        sh 'pwd; sleep 180; echo "Application Has been deployed on K8S"'
      }
      }
      
    stage('RunDASTUsingZAP') {
            steps {
          withKubeConfig([credentialsId: 'kubelogin']) {
          sh('zap.sh -cmd -quickurl http://$(kubectl get services/zdevsecops --namespace=devsecops -o json| jq -r ".status.loadBalancer.ingress[] | .hostname") -quickprogress -quickout ${WORKSPACE}/zap_report.html')
          archiveArtifacts artifacts: 'zap_report.html'
          }
        }
      } 
 }	
}
