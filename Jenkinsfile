pipeline {
  agent any
  stages {
    stage('Build') {
      steps {
        sh 'echo "Hello World"'
        sh '''
          echo "Multiline shell steps works too"
          ls -lah
        '''
      }
    }
    stage('Upload to AWS') {
      steps {
        withAWS(region: 'us-east-1', credentials: 'AKIAXJX6GQKY3PX5FDP6') {
          def identity=awsIdentity();//Log AWS credentials
          s3Upload(file: 'index.html', bucket: 'static-site-jenkins', path: './index.html')
        }
      }
    }
  }
}
