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
                  sh 'mvn clean compile sonar:sonar -Dsonar.host.url=http://138.91.197.195:9000 -Dsonar.sources=. -Dsonar.tests=. -Dsonar.inclusions=**/test/java/servlet/createpage_junit.java -Dsonar.test.exclusions=**/test/java/servlet/createpage_junit.java -Dsonar.login=admin -Dsonar.password=admin' 
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
        deploy adapters: [tomcat8(credentialsId: 'tomcat', path: '', url: 'http://40.88.14.229:8080')], contextPath: '/QAWebapp', war: '**/*.war'
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
        sh 'mvn test -f functionaltest/pom.xml'
        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '\\functionaltest\\target\\surefire-reports', reportFiles: 'index.html', reportName: 'UI-Test', reportTitles: ''])
      }
    }

    stage('Performance Test') {
      steps {
        echo 'Performance test'
        blazeMeterTest(credentialsId: 'blazemeter', workspaceId: '680689', testId: '8642591.taurus')
      }
    }

    stage('Deploy to Production') {
      steps {
        echo 'Deploy to Production'
        sh 'mvn clean install'
        deploy adapters: [tomcat8(credentialsId: 'tomcat', path: '', url: 'http://40.88.9.18:8080')], contextPath: '/ProdWebapp', war: '**/*.war'
        slackSend channel: '#devops', message: 'Code deployed to prod server'
      }
    }

    stage('Perform Sanity Check') {
      steps {
        echo 'Perform Sanity Check'
        sh 'mvn test'
        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '\\Acceptancetest\\target\\surefire-reports', reportFiles: 'index.html', reportName: 'Sanity Test report', reportTitles: ''])
        slackSend channel: '#devops', message: 'Sanity test completed successfully'
      }
    }

  }
}
