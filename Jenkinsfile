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
            withAWS(region: 'eu-west-2', credentials: 'afdb0fa4-bd8e-4f96-ba10-50d277e5b1d2') {
              s3Delete(bucket: 'zero2heros3bucket', path: '**/*')
              s3Upload(bucket: 'zero2heros3bucket', workingDir: 'build', includePathPattern: '**/*', acl:'BucketOwnerFullControl')
            }

            mail(subject: 'Production Build', body: 'New Deployment to Production', to: 'alper.atay@gmail.com')
            sh 'sh ./scripts/deliver-for-development.sh'
            input message: 'Finished using the web site? (Click "Proceed" to continue)', ok: 'continue'
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
            }

            mail(subject: 'Production Build', body: 'New Deployment to Production', to: 'alper.atay@gmail.com')
            sh 'sh ./scripts/deliver-for-development.sh'
            input message: 'Finished using the web site? (Click "Proceed" to continue)', ok: 'continue'
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
