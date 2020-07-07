pipeline {
  agent {
    docker {
      image 'node:10-alpine'
      args '-p 3000:3000 -p 8000:8000'
    }

  }
  stages {
    stage('Install Packages') {
      steps {
        sh 'npm install'
      }
    }

    stage('Test and Build') {
      parallel {
        stage('Run Tests') {
          steps {
            sh ' sh ./scripts/test.sh'
          }
        }

        stage('Create Build Artifacts') {
          steps {
            sh 'npm run build'
          }
        }

      }
    }

    stage('Deployment') {
      parallel {
        stage('Staging') {
          when {
            branch 'staging'
          }
          steps {
            withAWS(region: 'eu-west-2', credentials: '36b93a18-1bb6-46cd-b0a3-d43e5b8c880b') {
              s3Delete(bucket: 'zero2hero-s3-bucket3', path: '**/*')
              s3Upload(bucket: 'zero2hero-s3-bucket3', workingDir: 'build', includePathPattern: '**/*', acl:'BucketOwnerFullControl')
            }

            mail(subject: 'Production Build', body: 'New Deployment to Production', to: 'alper.atay@gmail.com')
            sh 'sh ./scripts/deliver-for-development.sh'
            sleep time: 100, unit: 'SECONDS'
            sh 'sh ./scripts/kill.sh'
          }
        }

        stage('Production') {
          when {
            branch 'master'
          }
          steps {
            withAWS(region: 'eu-west-2', credentials: '36b93a18-1bb6-46cd-b0a3-d43e5b8c880b') {
              s3Delete(bucket: 'zero2hero-s3-bucket3', path: '/sample-node-app/*')
            }

            mail(subject: 'Production Build', body: 'New Deployment to Production', to: 'alper.atay@gmail.com')
            sh 'sh ./scripts/deliver-for-development.sh'
            sleep time: 100, unit: 'SECONDS'
            sh 'sh ./scripts/kill.sh'
          }
        }

      }
    }

  }
  environment {
    CI = 'true'
    HOME = '.'
    npm_config_cache = 'npm-cache'
  }
}
