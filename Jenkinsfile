pipeline {

    options {
        buildDiscarder (
            logRotator (
                artifactDaysToKeepStr: '',
                artifactNumToKeepStr: '',
                daysToKeepStr: '',
                numToKeepStr: '1'
            )
        )
        disableConcurrentBuilds()
    }

    agent any

    stages {
        stage('Install dependencies') {
            agent {
                dockerfile {
                    filename 'Dockerfile.build'
                    args '-v ${PWD}:/usr/src/app -w /usr/src/app'
                    label 'build-image'
                    reuseNode true
                }
            }
            steps {
                sh 'yarn'
                sh 'cd src && yarn'
            }
        }


        stage('Run tests') {
            agent {
                dockerfile {
                    filename 'Dockerfile.build'
                    args '-v ${PWD}:/usr/src/app -w /usr/src/app'
                    label 'build-image'
                    reuseNode true
                }
            }
            steps {
                sh 'yarn test'
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
