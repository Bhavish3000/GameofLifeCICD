pipeline {
    agent any
    triggers {
        pollSCM('H/15 * * * *')
    }

    environment {
        CODEQL_RESULTS = 'results.sarif'
        GITHUB_REPO = 'Bhavish3000/GameofLifeCICD'
    }
    
    tools {
        maven 'maven3.9.9'
        jdk 'java8'
        codeql 'CodeQL2.20.7'
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

        stage('CodeQL Analysis') {
            steps {
                withCodeQL(codeql: 'CodeQL2.20.7') {
                      sh 'codeql database create codeql-db --language=java --source-root=. --overwrite'
                      sh "codeql database analyze codeql-db codeql/java-queries --format=sarif-latest --output=${CODEQL_RESULTS} --threads=4"
                }
            }
        }

        stage('Archive Results') {
            steps {
               archiveArtifacts artifacts: "${CODEQL_RESULTS}",
                fingerprint: true 
            }
        }

        stage('Upload Results to GitHub') {
            steps {
                withCredentials([string(credentialsId: 'GITHUB_TOKEN', variable: 'GITHUB_TOKEN')]) {
                   sh """
                        curl -X POST -H "Authorization: token ${GITHUB_TOKEN}" \
                        -H "Accept: application/vnd.github.v3+json" \
                        https://api.github.com/repos/${GITHUB_REPO}/code-scanning/sarif \
                        -d @${CODEQL_RESULTS}
                    """

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
        always {
           withCodeQL(codeql: 'CodeQL2.20.7') {
            sh 'codeql database cleanup codeql-db --force'
           }
        }
    }
}
