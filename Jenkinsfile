pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' // test  commit and push
            }
        }   
      stage('Tests') {
            steps {
              sh "mvn test"
            }
        }   
    }
}
