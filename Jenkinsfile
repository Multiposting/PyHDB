#!/usr/bin/groovy

boolean continuePipeline = false
def nodeLabel = 'ci-python-35'

jenkinsTemplate(nodeLabel, ['docker', 'python35']) {
    node(nodeLabel) {
        checkout scm

        stage('Install jenkins dependencies') {
            container('python') {
                sh 'pip install pytest mock'
            }
        }

        stage('Running tests') {
            container('python') {
                stage('Units tests') {
                    sh 'py.test'
                }
            }
        }
    }

    if (env.BRANCH_NAME == 'master') {
        continuePipeline = true
    }

    if(continuePipeline) {
        stage('Approval') {
            timeout(time:1, unit:'HOURS') {
                input message:'Approve upload on pypi?'
                continuePipeline = true
            }
        }

        node(nodeLabel) {
            checkout scm

            stage ('Building package') {
                container('python') {
                    sh 'pip install setuptools wheel twine'
                    sh 'python setup.py sdist bdist_wheel'
                }
            }

            stage ('Publish on PyPi') {
                withCredentials([file(credentialsId: '29f1b3af-5644-48b3-bffc-6ca34b1e0c3e', variable: 'pypirc')]) {
                    container('python') {
                        sh "twine upload --config-file $pypirc --repository internal dist/*"
                    }
                }
            }
        }
    }
}
