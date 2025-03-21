pipeline {
    agent any
    triggers {
        pollSCM('H/15 * * * *')
    }
    
    tools {
        maven 'maven3.9.9'
        jdk 'java8'
    }

    stages {
        
        stage('Checkout SCM') {
            steps{
                git url: 'https://github.com/Bhavish3000/game-of-life_fork.git',
                    branch: 'master'
            }

        }

        stage('Build') {
            steps {
                withMaven(
                    maven: 'maven3.9.9',
                    jdk: 'java8',
                    traceability: true
                ) {
                    sh 'mvn clean install'
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: '**/target/*.war',
                    fingerprint: true,
                    onlyIfSuccessful: true

            }
        }
    }

    post {
        success {
            echo 'Build and artifact archiving completed successfully!'
        }

        failure {
            echo 'Build or artifact archiving failed!'
        }
    }
}