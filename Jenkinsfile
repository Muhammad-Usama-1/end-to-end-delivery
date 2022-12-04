pipeline {
    agent any
    tools  {
        // This nodejs name should be same as you configure in jenkins
        nodejs "17.3.1"
    }

    stages {
        stage('Hello') {
            steps {
                sh "npm version"
            }
        }
        stage('CheckoutCode'){
            steps {
             git  url: 'https://github.com/Muhammad-Usama-1/nodejs-app-mss'
            }

        }
        stage("Building the Applcation") {
            steps {
                sh "npm install"
            }
        }

         stage('Executating the Sonarqube report') {
            steps {
                sh "npm run sonar"
            }
        }
    }
}