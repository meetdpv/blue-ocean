pipeline {
  agent any
  stages {
    stage('App_Build_ST') {
        steps {
			echo 'Build Number: ' + env.BUILD_NUMBER
			
				deleteDir()
				git(url: 'https://github.com/meetdpv/spring-petclinic.git', branch: 'master')
            	sh([script:"${tool 'M3'}/bin/mvn clean install -DskipTests"])
			
      	}
    }
    stage('Unit_Tests_ST') {
      steps {
		
        	echo 'Build Number: ' + env.BUILD_NUMBER
			sh([script:"${tool 'M3'}/bin/mvn test"])
			archiveArtifacts artifacts: '**/*' 
		
      }
    }
    stage('Code_Analysis_ST') {
      steps {
			echo 'Build Number: ' + env.BUILD_NUMBER
			build job: 'Extra_Jobs/Sonar_Code_Analysis', parameters: [string(name: 'B', value: BUILD_NUMBER)]
		}
    }
    stage('Deploy_Environment_ST') {
      steps {
		
			echo 'Build Number: ' + env.BUILD_NUMBER
						
			sh '''echo "
			FROM tomcat:8.0 
			ADD target/petclinic.war /usr/local/tomcat/webapps/
			" > ${WORKSPACE}/Dockerfile
			export REPO_NAME="$(echo ${JOB_NAME} | tr '/' '_' | tr '[:upper:]' '[:lower:]')"
			docker build -t ${REPO_NAME}/adop-foss-java:0.0.${BUILD_NUMBER} .
			echo "New image has been build - ${REPO_NAME}/adop-foss-java:0.0.${BUILD_NUMBER}"'''

		
      }
    }
    stage('Test_Build_ST') {
      steps {
		
			git(url: 'https://github.com/meetdpv/blueocean', branch: 'master')
			bat([script:"${tool 'M3'}/bin/mvn clean compile install -DskipTests"])
		
      }
    }
	stage('Continuous_Testing_ST') {

	steps {
	  parallel(
		"01-Functional": {
			echo 'Functional Testing...'
				bat([script:'set MAVEN_OPTS = –Xmx2048m'])
				bat([script:'mvn exec:java -X -Dexec.mainClass="com.accenture.runner.selenium.SELENIUM_Executor" -Dexec.classpathScope=test'])			
			},
          "02-Platform": {
            echo 'Platform Testing...'
				bat([script:'mvn exec:java -X -Dexec.mainClass="com.accenture.runner.platform.PLATFORM_Executor" -Dexec.classpathScope=test'])
			},
          "03-BDD": {
            echo 'BDD Testing...'
				bat([script:'mvn exec:java -X -Dexec.mainClass="com.accenture.runner.bdd.BDD_Executor" -Dexec.classpathScope=test'])
			},
          "04-API": {
            echo ' API Testing...'
				bat([script:'start /b mvn jetty:run'])
				bat([script:'mvn integration-test'])
				bat([script:'mvn jetty:stop'])
			}
        )
      }
    }
    stage('Pre-Prod-Deploy') {
      steps {
        echo 'Pre-Prod-Deployment started...'
      }
    }
  }
  post {
    always {
      echo 'Post Build Step'
    }
  }
}
