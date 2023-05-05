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
     stage('Docker Build and Push') {
      steps {
          withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
          sh 'printenv'
          sh 'docker build -t kittudodda23/numeric-app:""$GIT_COMMIT"" .'
          sh 'docker push kittudodda23/numeric-app:""$GIT_COMMIT""'
        }
      }
    }
     stage('Kubernetes Deployment - DEV') {
      steps {
        withKubeConfig([credentialsId: 'kubeconfig']) {
          sh "sed -i 's#replace#kittudodda23/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
          sh "kubectl apply -f k8s_deployment_service.yaml"
        }
      }
    } 
    stage('SonarQube - SAST') {
      steps {
        sh "mvn sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.host.url=http://devsecops-k8s.eastus.cloudapp.azure.com:9000 -Dsonar.login=15446940b79fa86cc4400fc252e67dfa97ed5df9"
      }
    }  
  }
}
