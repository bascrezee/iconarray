def BuildBadge = addEmbeddableBadgeConfiguration(id: "build", subject: "Build")
def TestBadge = addEmbeddableBadgeConfiguration(id: "test", subject: "Test")

pipeline {
    environment {
       EMAIL_TO_1 = 'victoria.cherkas@meteoswiss.ch'
       EMAIL_TO_2 = 'victoria.cherkas@meteoswiss.ch'
       CONDA_ENV_NAME = 'iconarray'
    }
    agent none
    stages {
        stage('Setup') {
            parallel {
                stage('setup miniconda on daint') {
                    agent { label 'daint' }
                    environment {
                        PATH = "$WORKSPACE/miniconda_$NODE_NAME/bin:$PATH"
                    }
                    steps {
                        script {
                            BuildBadge.setStatus('running')
                        }
                        sh 'wget -O ${WORKSPACE}/miniconda.sh https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh'
                        sh 'bash miniconda.sh -b -p $WORKSPACE/miniconda_${NODE_NAME}'
                        sh 'conda config --set always_yes yes --set changeps1 no'
                        sh 'conda config --add channels conda-forge'
                        sh 'conda env create --name ${CONDA_ENV_NAME}_${NODE_NAME} --file env/environment.yml'
                        sh '''source $WORKSPACE/miniconda_${NODE_NAME}/etc/profile.d/conda.sh
                            conda activate ${CONDA_ENV_NAME}_${NODE_NAME}
                            source env/setup-conda-env.sh
                            conda deactivate
                            rm miniconda.sh'''
                    }
                    post {
                        failure {
                            echo 'Cleaning up workspace'
                            deleteDir()
                        }
                    }
                }
                stage('setup miniconda on tsa') {
                    agent { label 'tsa' }
                    environment {
                        PATH = "$WORKSPACE/miniconda_$NODE_NAME/bin:$PATH"
                    }
                    steps {
                        script {
                            BuildBadge.setStatus('running')
                        }
                        sh 'wget -O ${WORKSPACE}/miniconda.sh https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh'
                        sh 'bash miniconda.sh -b -p $WORKSPACE/miniconda_${NODE_NAME}'
                        sh 'conda config --set always_yes yes --set changeps1 no'
                        sh 'conda config --add channels conda-forge'
                        sh 'conda env create --name ${CONDA_ENV_NAME}_${NODE_NAME} --file env/environment.yml'
                        sh '''source $WORKSPACE/miniconda_${NODE_NAME}/etc/profile.d/conda.sh
                            conda activate ${CONDA_ENV_NAME}_${NODE_NAME}
                            source env/setup-conda-env.sh
                            conda deactivate
                            rm miniconda.sh'''
                    }
                    post {
                        failure {
                            echo 'Cleaning up workspace'
                            deleteDir()
                        }
                    }
                }
            }
            post {
                failure {
                    node('tsa') {
                        script {
                            BuildBadge.setStatus('failing')
                        }
                    }
                }
                success {
                    node('tsa') {
                        script {
                            BuildBadge.setStatus('passing')
                        }
                    }
                }
            }
        }
        stage('Test') {
            parallel {
                stage('test on daint') {
                    agent { label 'daint' }
                    environment {
                        PATH = "$WORKSPACE/miniconda_${NODE_NAME}/bin:$PATH"
                    }
                    steps {
                        script {
                            TestBadge.setStatus('running')
                            sh '''source $WORKSPACE/miniconda_${NODE_NAME}/etc/profile.d/conda.sh
                            conda activate ${CONDA_ENV_NAME}_${NODE_NAME}
                            pip install .
                            python -m cfgrib selfcheck
                            python -c "import cartopy; print(cartopy.config)"
                            python iconarray/utils/get_data.py
                            pytest iconarray/tests'''
                        }
                    }
                    post {
                        success {
                            mail bcc: '',
                            body: "<b>Jenkins Success</b><br>Project: ${env.JOB_NAME}<br>Build Number: ${env.BUILD_NUMBER}<br>Build URL: ${env.BUILD_URL}" ,
                            cc: "${EMAIL_TO_2}", charset: 'UTF-8', from: '', mimeType: 'text/html',
                            replyTo: '', subject: "Jenkins Job Success ${NODE_NAME} ->${env.JOB_NAME}",
                            to: "${EMAIL_TO_1}";
                        }
                        failure {
                            script {
                                mail bcc: '',
                                body: "<b>Jenkins Failure</b><br>Project: ${env.JOB_NAME}<br>Build Number: ${env.BUILD_NUMBER}<br>Build URL: ${env.BUILD_URL}" ,
                                cc: "${EMAIL_TO_2}", charset: 'UTF-8', from: '', mimeType: 'text/html',
                                replyTo: '', subject: "Jenkins Job Failure ${NODE_NAME} -> ${env.JOB_NAME}",
                                to: "${EMAIL_TO_1}";
                            }

                        }
                        always {
                            archiveArtifacts artifacts: '*.log', allowEmptyArchive: true
                            echo 'Cleaning up workspace'
                            deleteDir()
                        }
                    }
                }
                stage('test on tsa') {
                    agent { label 'tsa' }
                    environment {
                        PATH = "$WORKSPACE/miniconda_${NODE_NAME}/bin:$PATH"
                    }
                    steps {
                        script {
                            TestBadge.setStatus('running')
                            sh '''source $WORKSPACE/miniconda_${NODE_NAME}/etc/profile.d/conda.sh
                            conda activate ${CONDA_ENV_NAME}_${NODE_NAME}
                            pip install .
                            python -m cfgrib selfcheck
                            python -c "import cartopy; print(cartopy.config)"
                            python iconarray/utils/get_data.py
                            pytest iconarray/tests'''
                        }
                    }
                    post {
                        success {
                            mail bcc: '',
                            body: "<b>Jenkins Success</b><br>Project: ${env.JOB_NAME}<br>Build Number: ${env.BUILD_NUMBER}<br>Build URL: ${env.BUILD_URL}" ,
                            cc: "${EMAIL_TO_2}", charset: 'UTF-8', from: '', mimeType: 'text/html',
                            replyTo: '', subject: "Jenkins Job Success ${NODE_NAME} ->${env.JOB_NAME}",
                            to: "${EMAIL_TO_1}";
                        }
                        failure {
                            mail bcc: '',
                            body: "<b>Jenkins Failure</b><br>Project: ${env.JOB_NAME}<br>Build Number: ${env.BUILD_NUMBER}<br>Build URL: ${env.BUILD_URL}" ,
                            cc: "${EMAIL_TO_2}", charset: 'UTF-8', from: '', mimeType: 'text/html',
                            replyTo: '', subject: "Jenkins Job Failure ${NODE_NAME} -> ${env.JOB_NAME}",
                            to: "${EMAIL_TO_1}";
                        }
                        always {
                            archiveArtifacts artifacts: '*.log', allowEmptyArchive: true
                            echo 'Cleaning up workspace'
                            deleteDir()
                        }
                    }
                }
            }
            post {
                failure {
                    node('tsa') {
                        script {
                            TestBadge.setStatus('failing')
                        }
                    }
                }
                success {
                    node('tsa') {
                        script {
                            TestBadge.setStatus('passing')
                        }
                    }
                }
            }
        }
    }
    post { 
        always { 
            node('tsa') {
                deleteDir()
            }
        }
    }
}
