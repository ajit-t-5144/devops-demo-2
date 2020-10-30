pipeline {
  agent any
 
  // environment variables
  environment {
    
    //System Variables
    buildnum = currentBuild.getNumber()
    
    //git repo details 
    gitURL = "https://github.com/ajit-t-5144/DevOps-Demo-WebApp.git"
    gitBranch = "*/master"
    
    //tomcat TEST Url and Path 
    tomcatTest = "http://13.82.213.187:8080"
    testPath = "/QAWebapp"
    
    //tomcat PROD URL and Path 
    tomcatProd = "http://13.82.211.112:8080"
    prodPath = "/ProdWebapp"
    
    
    // Sonarqube URL 
    sonarPath = 'http://13.64.108.228:9000'
    
    sonarInclusion = '**/test/java/servlet/createpage_junit.java'
    sonarExclusion = '**/test/java/servlet/createpage_junit.java'
    
    //UI retprt path
    uiPath = "\\functionaltest\\target\\surefire-reports" 
    
    //Sanity Report Path 
    sanityPath = "\\Acceptancetest\\target\\surefire-reports"
    
    //Slack Channel details
    sChannel = "#devops"
    
  }
  
  
  //Global tools 
  tools { 
        maven 'maven' 
        jdk  'jdk'
    }
  
  // Job stages
  stages {
    
    //Initiating the Jenkins Build 
    stage ('Initiation') {
      
      steps {
        slackSend channel: "${sChannel}", message: 'Starting Jenkins Build for Devops Web App . Build Number:' + "${buildnum}"
      }
    }
    // Static code analysis - inject sonarqube 
    stage('Static-analysis') {
      steps {
        echo 'Static code Analysis'
        checkout([$class: 'GitSCM', branches: [[name: "${gitBranch}"]], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: "${gitURL}"]]])
        withSonarQubeEnv(credentialsId: 'sonar', installationName: 'sonarqube')
        {sh 'mvn clean compile sonar:sonar -Dsonar.host.url=${sonarPath} -Dsonar.sources=. -Dsonar.tests=. -Dsonar.inclusions=${sonarInclusion} -Dsonar.test.exclusions=${sonarExclusion} -Dsonar.login=admin -Dsonar.password=admin' 
            }
        slackSend channel: '#devops', message: 'Stattic test analysis completed'
      }
    } // Stage end

    
   //Compile the Web app 
    stage('Compile-Build') {
      steps {
        echo 'Build the code'
        sh 'mvn clean compile'
      }
    }
  
    //Deploy the application to tomcat Test Server 
    stage('Deploy To Test') {
      steps {
        echo 'Deploy to Test'
        sh 'mvn clean package'
        deploy adapters: [tomcat8(credentialsId: 'tomcat', path: '', url: "${tomcatTest}")], contextPath: "${testPath}", war: '**/*.war'
        slackSend channel: "${sChannel}", message: 'Code deployed to Test Server'
      }
    }
    
    //Store artifact and Build info in artifactory server
    stage('Store Artifact') {
      steps {
        echo 'Store Artifact' 
        rtUpload(serverId: 'artifactory')
        rtPublishBuildInfo (serverId: 'artifactory')
        slackSend channel: "${sChannel}", message: 'Artifacts deployed to Artifactory'
      }
    }

    //Perform UI Test and Publish the report
    stage('Perform UI Test') {
      steps {
        echo 'UI Test'
        sh 'mvn test -f functionaltest/pom.xml'
        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: "${uiPath}", reportFiles: 'index.html', reportName: 'UI-Test', reportTitles: ''])
        slackSend channel: "${sChannel}", message: 'UI Test Completed Successfully'
      }
    }
   
    //Run Performance Test using Blazemeter
    stage('Performance Test') {
      steps {
        echo 'Performance test'
        //blazeMeterTest(credentialsId: 'blazemeter', workspaceId: '680689', testId: '8642591.taurus')
      }
    }

    //Deploy the Webapp to Production tomcat Server 
    stage('Deploy to Production') {
      steps {
        echo 'Deploy to Production'
        sh 'mvn clean install'
        deploy adapters: [tomcat8(credentialsId: 'tomcat', path: '', url: "${tomcatProd}")], contextPath: "${prodPath}", war: '**/*.war'
        slackSend channel: "${sChannel}", message: 'Code deployed to prod server'
      }
    }
    
    //Run Sanity Check and publish the report  
    stage('Perform Sanity Check') {
      steps {
        echo 'Perform Sanity Check'
        sh 'mvn test'
        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: "${sanityPath}", reportFiles: 'index.html', reportName: 'Sanity Test report', reportTitles: ''])
        slackSend channel: "${sChannel}", message: 'Sanity test completed successfully'
      }
    }
    
    stage ('Completion') {
      
      steps {
        slackSend channel: "${sChannel}", message: 'jenkins Build ' + "${buildnum}" + ' completed Successfully'
      }
    }
    
  } //Job stages end 
} // end of Pipeline
