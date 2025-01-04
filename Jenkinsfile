pipeline {
    agent {
        node {
            label 'maven'
        }
      
    }
    stages {
        stage(clone) {
            steps{
                git branch: 'main', url: 'https://github.com/Sathish-kumar0/tweet-trend-new.git'
            }
        }
    }
}