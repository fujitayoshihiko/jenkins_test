pipeline {
  agent any
  stages {
    stage('test1') {
      steps {
        echo 'test'
        retry(count: 5) {
          withCredentials(bindings: [sshUserPrivateKey(credentialsId: 'cb4692c9-04ae-47c8-b0de-869adadb9466', keyFileVariable: 'sshkey', passphraseVariable: '', usernameVariable: '')]) {
          sh '''ls
hostname
echo "end"
ssh -o "StrictHostKeyChecking=no" -i $sshkey root@192.168.86.100 ls
'''
          }
        }
      }
    }

  }
}
