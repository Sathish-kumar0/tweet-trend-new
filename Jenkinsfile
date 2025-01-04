def registry = 'https://trial7aanip.jfrog.io'
pipeline {
    agent {
        node {
            label 'maven'
        }
    }
    environment {
        PATH = "/opt/apache-maven-3.9.9/bin:$PATH"
    }
    stages {
        stage ('build') {
            steps {
                echo"-------------build started------------"
                sh 'mvn clean deploy -DskipTests'
                echo"-------------build started------------"
            }
        }
      /*   stage ("test") {
            steps{
                echo "-------------unit test started------------"
                sh 'mvn surefire-report:report'
                echo "-------------unit test completed------------"
            }
        }*/
        stage('SonarQube analysis') {
            environment{
                scannerHome = tool 'valaxy-sonar-scanner';
            }
            steps{
                withSonarQubeEnv('valaxy-sonarqube-server') { // If you have configured more than one global server connection, you can specify its name
      sh "${scannerHome}/bin/sonar-scanner"
    }
            }
            
        }
        stage("Quality Gate"){
            steps{
                script{
                    timeout(time: 1, unit: 'HOURS') { // Just in case something goes wrong, pipeline will be killed after a timeout
    def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
    if (qg.status != 'OK') {
      error "Pipeline aborted due to quality gate failure: ${qg.status}"
    }
  }
                }
                

            }
  
}

        stage("JFROG Articat") {
                 
         stage("Jar Publish") {
        steps {
            script {
                    echo '<--------------- Jar Publish Started --------------->'
                     def server = Artifactory.newServer url:registry+"/artifactory" ,  credentialsId:"jfrog-cred"
                     def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}";
                     def uploadSpec = """{
                          "files": [
                          {
                          "pattern": "jarstaging/(*)",
                          "target": "libs-release-local/{1}",
                          "flat": "false",
                          "props" : "${properties}",
                         
                          
                          }
                          ]
                     }"""
                     def buildInfo = server.upload(uploadSpec)
                     buildInfo.env.collect()
                     server.publishBuildInfo(buildInfo)
                     echo '<--------------- Jar Publish Ended --------------->'  
            
            }
        }
        }
    }
}
}