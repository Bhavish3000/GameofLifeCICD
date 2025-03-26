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
                    branch: 'master',
                    credentialsId: 'GithubCredentials'
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

        stage('SonarCloud analysis') {
            steps {
                withSonarQubeEnv(credentialsId: 'SONARCLOUD_TOKEN',installationName: 'SONAR_CLOUD') {
                    sh '''
                    mvn clean package \
                    org.sonarsource.scanner.maven:sonar-maven-plugin:3.7.0.1746:sonar \
                    -Dsonar.organization=gameoflifebhavish \
                    -Dsonar.projectKey=your-project-key-here
                    '''
                        }  
                          
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
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
