pipeline {
  agent any
  //define artifactserver = artifactory.server('ajdevopstcs1.jfrog.io')
  tools { 
        maven 'maven' 
        jdk  'jdk'
    }
  stages {
    stage('Static-analysis') {
      steps {
        echo 'Static code Analysis'
        checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/ajit-t-5144/DevOps-Demo-WebApp.git']]])
        //waitForQualityGate(abortPipeline: true, credentialsId: 'sonarqube', installationName: 'sonarqube')
        withSonarQubeEnv(credentialsId: 'sonar', installationName: 'sonarqube')
           {    withMaven{
                  //sh 'maven $SONAR_MAVEN_GOAL -Dsonar.host.url=$SONAR_HOST_URL'
                  //sh 'mvn -Dsonar.test.exclusions=**/test/java/servlet/createpage_junit.java -Dsonar.login=admin -Dsonar.password=admin -Dsonar.tests=. -Dsonar.inclusions=**/test/java/servlet/createpage_junit.java -Dsonar.sources=. sonar:sonar -Dsonar.host.url=http://35.193.147.208:9000'
                  sh 'mvn clean compile sonar:sonar -Dsonar.host.url=http://104.42.72.53:9000 -Dsonar.sources=. -Dsonar.tests=. -Dsonar.inclusions=**/test/java/servlet/createpage_junit.java -Dsonar.test.exclusions=**/test/java/servlet/createpage_junit.java -Dsonar.login=admin -Dsonar.password=admin' 
                  }
                }
        slackSend channel: '#devops', message: 'Stattic test analysis completed'
      }
    }

    stage('Compile-Build') {
      steps {
        echo 'Build the code'
        sh 'mvn clean compile'
      }
    }

    stage('Deploy To Test') {
      steps {
        echo 'Deploy to Test'
        sh 'mvn clean package'
        deploy adapters: [tomcat8(credentialsId: 'tomcat', path: '', url: 'http://52.142.4.251:8080')], contextPath: '/QAWebapp', war: '**/*.war'
        slackSend channel: '#devops', message: 'Code deployed to Test Server'
      }
    }
    
    
    stage('Store Artifact') {
      steps {
        echo 'Store Artifact' 
        rtUpload(serverId: 'artifactory')
        rtPublishBuildInfo (serverId: 'artifactory')
      }
    }

    stage('Perform UI Test') {
      steps {
        echo 'UI Test'
      }
    }

    stage('Performance Test') {
      steps {
        echo 'Performance test'
      }
    }

    stage('Deploy to Production') {
      steps {
        echo 'Deploy to Production'
      }
    }

    stage('Perform Sanity Check') {
      steps {
        echo 'Perform Sanity Check'
      }
    }

  }
}
