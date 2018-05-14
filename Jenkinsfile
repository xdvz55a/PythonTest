pipeline {
    agent none 
    stages {
        stage('Build') { 
            agent {
                docker {
                    image 'python:2-alpine' 
                }
            }
            steps {
                sh 'python -m py_compile project/sources/add2vals.py project/sources/calc.py' 
            }
        }
        stage('Nexus Lifecycle Analysis') {
              postGitHub commitId, 'pending', 'analysis', 'Nexus Lifecycle Analysis is running'

              try {
                def policyEvaluation = nexusPolicyEvaluation iqApplication: 'sandbox-application', iqStage: 'release'
                postGitHub commitId, 'success', 'analysis', 'Nexus Lifecycle Analysis succeeded', "${policyEvaluation.applicationCompositionReportUrl}"
              } catch (error) {
                def policyEvaluation = error.policyEvaluation
                postGitHub commitId, 'failure', 'analysis', 'Nexus Lifecycle Analysis failed', "${policyEvaluation.applicationCompositionReportUrl}"
                throw error
              }
            }
        stage('Test') {
            agent {
                docker {
                    image 'qnib/pytest'
                }
            }
            steps {
                sh 'py.test --verbose --junit-xml test-reports/results.xml project/sources/test_calc.py'
            }
            post {
                always {
                    junit 'test-reports/results.xml'
                }
            }
        }
        stage('Deliver') {
            agent {
                docker {
                    image 'cdrx/pyinstaller-linux:python2'
                }
            }
            steps {
                sh 'pyinstaller --onefile project/sources/add2vals.py'
            }
            post {
                success {
                    archiveArtifacts 'dist/add2vals'
                }
            }
        }
    }
}
