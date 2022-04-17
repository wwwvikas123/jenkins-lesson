pipeline {
    agent any 
    environment {
     image = docker.build("unittests:${env.BRANCH_NAME}")
    }
    options {
      parallelsAlwaysFailFast()
    }

    stages {

        stage('Checkout code') { 
            steps {
                checkout scm
            }
            
        }        
        stage('tests') {
            parallel {
                stage('Run unittests') {
                    steps {
                        script {
                            image.inside {
                                sh "python3 -m pytest"
                            }
                        }
                    }
                }
                stage('Run linter') {
                    steps {    
                        script {
                            try{
                            image.inside() {
                                sh "prospector -o pylint:report/pylint.log src/"
                            }
                        
                            } catch (error) {
                                throw error
                            } finally {
                                sh "cat report/pylint.log"
                            }
                        }
                    }
                }
            }
        }
        stage('Publish reports') { 
            steps {
                cobertura  (
                onlyStable: false,
                enableNewApi: true,
                failUnhealthy: false,
                failUnstable: false,
                autoUpdateHealth: false,
                autoUpdateStability: false,
                zoomCoverageChart: false,
                maxNumberOfBuilds: 0,
                sourceEncoding: 'ASCII',
                coberturaReportFile: 'report/coverage.xml',
                lineCoverageTargets: '80, 0, 0',
                methodCoverageTargets: '80, 0, 0',
                conditionalCoverageTargets: '70, 0, 0'
                )
            }
        }
        stage('Build') { 
            steps {
                script {
                    image.inside {
                        sh 'python3 -m build'
                    }
                }
            }
        }
        stage('Publish artifact') { 
            steps {
                archiveArtifacts artifacts: 'dist/*.whl', fingerprint: false
            }
        }
    }        
}
