pipeline {
  agent any
  stages {
    stage('test1') {
      steps {
        echo 'test'
        retry(count: 5) {
          withCredentials(bindings: [sshUserPrivateKey(credentialsId: 'cb4692c9-04ae-47c8-b0de-869adadb9466', keyFileVariable: 'sshkey')]) {
            sh '''
ssh -o "StrictHostKeyChecking=no" -i $sshkey root@192.168.86.100 <<\'EOF\'

LOG_FILE=/var/log/nginx/access_bid.log

ls /var/log/nginx
ls ${LOG_FILE}

if [ ! -r ${LOG_FILE} ]; then
    echo "unable to read nginx log file(${LOG_FILE})"
    exit 1
fi

LAST_LOG_EPOCH=$(date -d "`tac ${LOG_FILE} | grep -v \'remote_addr:10\\.[1-3]\' | head -n 1 | sed -e \'s/.*\\ttime:\\([^[:space:]]\\+\\).*/\\1/\' -e \'s/:/ /\' -e \'s/\\// /g\'`" +%s)
NOW_EPOCH=`date +%s`

if [[ $(($NOW_EPOCH - $LAST_LOG_EPOCH)) -gt 60 ]]; then
    exit 0
fi

echo "one minute has not passed($(($NOW_EPOCH - $LAST_LOG_EPOCH)))"
exit 1
EOF
'''
          }

        }

      }
    }

  }
}
