pipeline {
  agent any
  stages {
    stage('test1') {
      steps {
        echo 'test'
        retry(count: 5) {
          
          withCredentials(bindings: [sshUserPrivateKey(credentialsId: 'cb4692c9-04ae-47c8-b0de-869adadb9466', keyFileVariable: 'sshkey', passphraseVariable: '', usernameVariable: '')]) {
            sh '''
LOG_FILE=/var/log/nginx/access_bid.log

if [ ! -r ${LOG_FILE} ]; then
    echo "LOGファイル(${LOG_FILE})を読み取れません"
    exit 0
fi

LAST_LOG_EPOCH=$(date -d "`tac ${LOG_FILE} | grep -v 'remote_addr:10\\.[1-3]' | head -n 1 | sed -e 's/.*\\ttime:\\([^[:space:]]\\+\\).*/\\1/' -e 's/:/ /' -e 's/\\// /g'`" +%s)
NOW_EPOCH=`date +%s`

if [[ $(($NOW_EPOCH - $LAST_LOG_EPOCH)) -gt 60 ]]; then
    exit 0
fi

echo "$(($NOW_EPOCH - $LAST_LOG_EPOCH))秒前にbidにリクエストがありました"
exit 1
EOF
'''
          }

        }

      }
    }

  }
}
