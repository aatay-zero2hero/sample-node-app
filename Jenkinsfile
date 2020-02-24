pipeline {
  agent {
    docker {
      image 'node:10-alpine'
      args '-p 20001-20100:3000'
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
            withAWS(region: 'eu-west-2', credentials: 'afdb0fa4-bd8e-4f96-ba10-50d277e5b1d2') {
              s3Delete(bucket: 'zero2heros3bucket', path: '**/*')
              s3Upload(bucket: 'zero2heros3bucket', workingDir: 'build', includePathPattern: '**/*')
            }

            mail(subject: 'Production Build', body: 'New Deployment to Production', to: 'alper.atay@gmail.com')
            sh 'sh ./scripts/deliver-for-development.sh'
            input 'Finished using the web site? (Click "Proceed" to continue)'
            sh 'sh ./scripts/kill.sh'
          }
        }

        stage('Production') {
          when {
            branch 'master'
          }
          steps {
            withAWS(region: 'eu-west-2', credentials: 'afdb0fa4-bd8e-4f96-ba10-50d277e5b1d2') {
              s3Delete(bucket: 'zero2heros3bucket', path: '/sample-node-app/*')
              s3Upload(bucket: 'zero2heros3bucket', workingDir: 'build', includePathPattern: '/sample-node-app/*', acl:'BucketOwnerFullControl')
            }

            mail(subject: 'Production Build', body: 'New Deployment to Production', to: 'alper.atay@gmail.com')
            sh 'sh ./scripts/deliver-for-development.sh'
            input 'Finished using the web site? (Click "Proceed" to continue)'
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
