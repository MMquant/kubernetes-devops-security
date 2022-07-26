pipeline {
  agent any

  stages {
    stage('Build Artifact - Maven') {
      steps {
        sh "mvn clean package -DskipTests=true"
        archive 'target/*.jar'
      }
    }

    stage('Unit Tests - JUnit and Jacoco') {
      steps {
        sh "mvn test"
      }
      post {
        always {
          junit 'target/surefire-reports/*.xml'
          jacoco execPattern: 'target/jacoco.exec'
        }
      }
    }
	stage('Mutation Tests - PIT') {
	      steps {
		sh "mvn org.pitest:pitest-maven:mutationCoverage"
	      }
	      post {
		always {
		  pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
		}
	      }
    }

	stage('SonarQube - SAST') {
	      steps {
		sh "mvn sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.host.url=http://3.70.10.172:9000 -Dsonar.login=f3ac7349143d51b0e98a1923f04902ecf6749baf"
	      }
      }

	stage('Docker Build and Push') {
	      steps {
		withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
		  sh 'printenv'
		  sh 'docker build -t mmquant/numeric-app:""$GIT_COMMIT"" .'
		  sh 'docker push mmquant/numeric-app:""$GIT_COMMIT""'
		}
	      }
	    }

	stage('Kubernetes Deployment - DEV') {
	      steps {
		withKubeConfig([credentialsId: 'kubeconfig']) {
		  sh "sed -i 's#replace#mmquant/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
		  sh "kubectl apply -f k8s_deployment_service.yaml"
        }
      }
    }
  }
}
