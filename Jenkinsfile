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
	
}
