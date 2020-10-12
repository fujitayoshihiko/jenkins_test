pipeline {
  agent any
  stages {
    stage('test1') {
      steps {
        echo 'test'
        withCredentials([sshUserPrivateKey(credentialsId: 'cb4692c9-04ae-47c8-b0de-869adadb9466', keyFileVariable: 'sshkey', passphraseVariable: '', usernameVariable: '')]) {
              // some block

                  sh '''ls
          hostname
          echo "end"'''
        }
      }
    }
  }
}
