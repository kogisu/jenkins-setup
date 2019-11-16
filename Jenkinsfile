pipeline {
  agent any
  stages {
    stage('Lint HTML') {
      steps {
        sh 'tidy -q -e *.html'
      }
    }
    stage('Upload to AWS') {
      steps {
        sh 'echo "Hello World"'
        sh '''
          echo "Multiline shell steps works too"
          ls -lah
        '''
        withAWS(region: 'us-east-1', credentials: 'aws-static') {
          s3Upload(file: 'index.html', bucket: 'static-site-jenkins', path: 'index.html')
        }
      }
    }
    
  }
}
