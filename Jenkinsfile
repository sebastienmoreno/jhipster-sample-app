node {
  stage('Checkout project') { 
    checkout scm
  }

  stage('build app'){
    docker.image('jhipster/jhipster').inside('--network ci') {
      sh './mvnw package -Pprod'
      step([$class: 'JUnitResultArchiver', allowEmptyResults: true, healthScaleFactor: 20, testResults: '**/target/surefire-reports/*.xml'])
    }
  }

  if (env.BRANCH_NAME ==~ 'master|develop|release-.*') {
    stage('push package to repository'){
      docker.image('maven:3.6.0-jdk-8-alpine').inside('--network ci') {
        sh './mvnw deploy -DaltDeploymentRepository=nexus-snapshots::default::http://nexus:8081/repository/maven-snapshots/'
      }
    }

    stage('build docker image'){
        sh 'docker build -t jhipstersampleapplication .'
    }

    stage('push image to dockerhub'){
      withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: "dockerhub", usernameVariable: 'DOCKER_HUB_USER', passwordVariable: 'DOCKER_HUB_PASSWORD']]) {
        sh 'docker login -u $DOCKER_HUB_USER -p $DOCKER_HUB_PASSWORD'
        sh 'docker tag jhipstersampleapplication $DOCKER_HUB_USER/jhipster-sample-app:latest'
        sh 'docker push $DOCKER_HUB_USER/jhipster-sample-app:latest'
      }
    }

    stage('SonarQube analysis') {
      docker.image('maven:3.6.0-jdk-8-alpine').inside('--network ci') {
        withSonarQubeEnv('sonarqube') {
          sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.2:sonar'
        }
      }
    }
  }
}
