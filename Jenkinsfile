pipeline {
    // 스테이지 별로 다른 거
    agent any

          triggers {
            // # ┌───────────── min (0 - 59)
            // # │   ┌────────────── hour (0 - 23)
            // # │   │ ┌─────────────── day of month (1 - 31)
            // # │   │ │ ┌──────────────── month (1 - 12)
            // # │   │ │ │ ┌───────────────── day of week (0 - 6) (0 to 6 are Sunday to Saturday, or use names; 7 is Sunday, the same as 0)
            // # │   │ │ │ │
            // # │   │ │ │ │
            // # *   * * * *  command to execute
        pollSCM('*/3 * * * *') // cron syntax
    }

    environment {
      // AWS의 Jenkins IAM의 권한을 가질 수 있게하기 위함.
      AWS_ACCESS_KEY_ID = credentials('awsAccessKeyId')
      AWS_SECRET_ACCESS_KEY = credentials('awsSecretAccessKey')
      AWS_DEFAULT_REGION = 'ap-northeast-2'
      HOME = '.' 
    }

    stages {
        // 레포지토리를 다운로드 받음. '준비' 단계
        stage('Prepare') {
            // 아무 node나
            agent any
            
            steps {
                // echo 'Lets start Long Journey! ENV: ${ENV}' -> ENV의 환경변수를 보여주기, 현재는 지정 안 되어 있어서 주석처리함.
                echo 'Clonning Repository'

                git url: 'https://github.com/HC-kang/jenkinsTest.git',
                    branch: 'master',
                    credentialsId: 'Jenkins'
                    // ㄴ여기도 Jenkins의 credential. jenkins의 토큰 필요
            }

            post {
                // 성공시?
                success {
                    echo 'Successfully Cloned Repository'
                }
                // 항상
                always {
                  echo "i tried..."
                }
                // 완료 후
                cleanup {
                  echo "after all other post condition"
                }
            }
        }
        
        // aws s3 에 파일을 올림
        stage('Deploy Frontend') {
          steps {
            echo 'Deploying Frontend'
            // 프론트엔드 디렉토리의 정적파일들을 S3 에 올림, 이 전에 반드시 EC2 instance profile 을 등록해야함.
            dir ('./website'){
                sh '''
                aws s3 sync ./ s3://jenkins-ford10082
                '''
            }
          }

          post {
              success {
                  echo 'Successfully Cloned Repository'
                  
                  // 성공하면 메일을 보내게 설정
                  mail  to: 'weston0713@gmail.com',
                        subject: "Deploy Frontend Success",
                        body: "Successfully deployed frontend!"

              }

              failure {
                  echo 'I failed :('

                  mail  to: 'weston0713@gmail.com',
                        subject: "Failed Pipelinee",
                        body: "Something is wrong with deploy frontend"
              }
          }
        }
        
        stage('Lint Backend') {
            // Docker plugin and Docker Pipeline 두개를 깔아야 사용가능!
            agent {
              docker {
                image 'node:latest'
              }
            }
            
            steps {
              dir ('./server'){
                  sh '''
                  npm install&&
                  npm run lint
                  '''
              }
            }

            post {
              // If Maven was able to run the tests, even if some of the test
              // failed, record the test results and archive the jar file.
              success {
                  echo 'Successfully Cloned Repository'

                  mail  to: 'weston0713@gmail.com',
                        subject: "LintBackend Success",
                        body: "Successfully deployed frontend!"

              }

              failure {
                  echo 'I failed :('

                  mail  to: 'weston0713@gmail.com',
                        subject: "Failed Pipelinee",
                        body: "Something is wrong with linting Backend"
              }
          }
        }
        
        stage('Test Backend') {
          agent {
            docker {
              image 'node:latest'
            }
          }
          steps {
            echo 'Test Backend'

            dir ('./server'){
                sh '''
                npm install
                npm run test
                '''
            }
          }
        }
        
        stage('Bulid Backend') {
          agent any
          steps {
            echo 'Build Backend'

            dir ('./server'){
                sh """
                docker build . -t server
                """
                // docker build . -t server --build-arg env=${PROD} -> 여기도 Production 환경변수가 지정되어있지 않아 제거.
            }
          }

          post {
            failure {
              error 'This pipeline stops here...'
            }
          }
        }
        
        stage('Deploy Backend') {
          agent any

          steps {
            echo 'Build Backend'

            dir ('./server'){
                // docker rm -f $(docker ps -aq)
                sh '''
                docker run -p 80:80 -d server
                '''
            } // rm -f 의 경우 이전 컨테이너를 죽이는 코드이니, 첫 빌드에서는 지워줘야 함! 
          }

          post {
            success {
              mail  to: 'weston0713@gmail.com',
                    subject: "Deploy Success",
                    body: "Successfully deployed!"
                  
            }
          }
        }
    }
}
