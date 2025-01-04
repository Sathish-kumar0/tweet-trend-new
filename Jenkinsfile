def registry = 'https://trial7aanip.jfrog.io'
def imageName = 'trial7aanip.jfrog.io/valaxy-docker-local//ttrend'
def version   = '2.1.2'
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

           
    stage(" Docker Build ") {
      steps {
        script {
           echo '<--------------- Docker Build Started --------------->'
           app = docker.build(imageName+":"+version)
           echo '<--------------- Docker Build Ends --------------->'
        }
      }
    }

            stage (" Docker Publish "){
        steps {
            script {
               echo '<--------------- Docker Publish Started --------------->'  
                docker.withRegistry(registry, 'jfrog-cred'){
                    app.push()
                }    
               echo '<--------------- Docker Publish Ended --------------->'  
            }
        }
    }
}
}