pipeline {
    agent any

    triggers {
        pollSCM('*/3 * * * *')
    }

    environment {
        AWS_ACESS_KEY_ID = credentials('awsAccessKeyId')
        AWS_SECRET_ACCESS_KEY = credentials('awsSecretAccessKey')
        AWS_DEFAULT_REGION = 'ap-northeast-2'
        HOME = '.' // Aviod npm root owned
    }

    stages {
        stage('Prepare') {
            
            agent any

            steps {
                // echo "lets start Long Journey! ENV: ${ENV}"
                echo "Clonning Repository"

                git url: 'https://github.com/HC-kang/jenkinsTest.git',
                branch: 'master',
                credentialsId: 'gitTokenForJenkins_HC-kang'
            }

            post {
                success {
                    echo 'Successfully pulled Repository'
                }

                always {
                    echo 'I tried'
                }

                cleanup {
                    echo 'after all other post condition'
                }
            }
        }

        // stage('Only for production') {
        //     when {
        //         branch 'production'
        //         environment name: 'APP_ENV', value: 'prod'
        //         anyOf {
        //             environment name: 'DEPOLY_TO', value: 'production'
        //             environment name: 'DEPOLY_TO', value: 'staging'
        //         }
        //     }
        // }

        stage('Deploy Fronted') {
            steps {
                echo 'Deploying Frontend'
                dir ('./website') {
                    sh '''
                    aws s3 sync ./ s3://jenkins-ford10082
                    '''
                }
            }

            post {
                success {
                    echo 'Successfully Pulled Repository'

                    mail  to: 'weston0713@gmail.com',
                          subject: "Deploy Frontend Success",
                          body: "Something is wrong with deploy fronted"
                }
            }

            failure {
                echo 'I failed...'

                mail  to: 'weston0713@gmail.com',
                      subject: 'Failed Pipelinee',
                      body: 'Something is wrong with deploy frontend'
            }
        }

        stage('Lint Backend') {
            agent {
                docker {
                    image 'node:latest'
                }
            }

            steps {
                dir ('./server') {
                    sh '''
                    npm install&&
                    npm run lint
                    '''
                }
            }
        }

        stage('TestBackend') {
            agent {
                docker {
                    image 'node:latest'
                }
            }
            steps {
                echo 'Test Backend'

                dir ('./server') {
                    sh '''
                    npm install
                    npm run test
                    '''
                }
            }
        }

        stage('Build Backend') {
            agent any
            steps {
                echo 'Build Backend'

                dir ('./server') {
                    sh '''
                    docker build . -t server --build-arg env=${PROD}
                    '''
                }
            }

            post {
                failure {
                    error 'This pipeline stops here'
                }
            }
        }

        stage('Deploy Backend') {
            agent any

            steps {
                echo 'Build Backend'

                dir ('./server') {
                    sh '''
                    docker run -p 80:80 -d server
                    '''
                    // docker rm -f $(docker ps -aq)
                }
            }

            post {
                success {
                    mail  to: 'weston0713@gmail.com',
                          subject: "Deploy Success",
                          body: "Successfully deployed"
                }
            }
        }

    }
}