pipeline {
  agent any
  stages {
    stage('server select') {
      input {
        message 'Should we continue?'
        id 'Yes, we should.'
        submitter 'alice,bob'
        parameters {
          string(name: 'Test', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')
        }
      }
      environment {
        Test = 'chose bid server!'
      }
      steps {
        echo "ok"
      }
    }

    stage('check lb-sout') {
      steps {
        echo "${Test}"
        retry(count: 30) {
          withCredentials(bindings: [sshUserPrivateKey(credentialsId: 'cb4692c9-04ae-47c8-b0de-869adadb9466', keyFileVariable: 'sshkey')]) {
            sh '''ssh -o "StrictHostKeyChecking=no" -i $sshkey root@192.168.86.100 <<\'EOF\'

LOG_FILE=/var/log/nginx/access_bid.log

if [ ! -r ${LOG_FILE} ]; then
    echo "unable to read nginx log file(${LOG_FILE})"
    sleep 60
    exit 1
fi

LAST_LOG_EPOCH=$(date -d "`tac ${LOG_FILE} | grep -v \'remote_addr:10\\.[1-3]\' | head -n 1 | sed -e \'s/.*\\ttime:\\([^\\t ]\\+\\).*/\\1/\' -e \'s/:/ /\' -e \'s/\\\\// /g\'`" +%s)
NOW_EPOCH=`date +%s`

if [[ $(($NOW_EPOCH - $LAST_LOG_EPOCH)) -gt 60 ]]; then
    echo "$(($NOW_EPOCH - $LAST_LOG_EPOCH)) second has pass"
    exit 0
fi

echo "one minute has not passed($(($NOW_EPOCH - $LAST_LOG_EPOCH)))"
sleep 60
exit 1

EOF'''
          }

        }

      }
    }

    stage('check log transferred') {
      steps {
        retry(count: 60) {
          withCredentials(bindings: [sshUserPrivateKey(credentialsId: 'cb4692c9-04ae-47c8-b0de-869adadb9466', keyFileVariable: 'sshkey')]) {
            sh '''ssh -o "StrictHostKeyChecking=no" -i $sshkey root@192.168.86.100 <<\'EOF\'
exit 0
BID=bid433
diff <(ssh localhost "find /fout/log/ -mtime -1 | grep -e \'extract\' | grep -e \'$BID\' | sort -n") <(ssh log11 "find /fout/log/ -mtime -1 | grep -e "extract" | grep -e \'$BID\' | sort -n")
RET=$?
if [ $RET -ne 0 ]; then
    sleep 60
    exit 1
fi

exit 0

EOF
'''
          }

        }

      }
    }

    stage('source commit') {
      input {
        message 'Did you commit the source?'
        id 'Yes'
      }
      steps {
        echo 'source commited!'
      }
    }

    stage('check td-agent log transfered') {
      steps {
        retry(count: 60) {
          withCredentials(bindings: [sshUserPrivateKey(credentialsId: 'cb4692c9-04ae-47c8-b0de-869adadb9466', keyFileVariable: 'sshkey')]) {
            sh '''ssh -o "StrictHostKeyChecking=no" -i $sshkey root@192.168.86.100 <<\'EOF\'
if [ `find /var/lib/td-agent/buffer | wc -l` -eq 1 ]; then
  exit 0
fi
find /var/lib/td-agent/buffer
sleep 10

exit 1
EOF
'''
          }

        }

      }
    }

    stage('stop services') {
      parallel {
        stage('stop service') {
          steps {
            withCredentials(bindings: [sshUserPrivateKey(credentialsId: 'cb4692c9-04ae-47c8-b0de-869adadb9466', keyFileVariable: 'sshkey')]) {
              sh '''ssh -o "StrictHostKeyChecking=no" -i $sshkey root@192.168.86.100 <<\'EOF\'
sudo systemctl stop td-agent
sudo systemctl stop svscan
sudo systemctl stop nginx

sudo systemctl disable td-agent
sudo systemctl disable svscan
sudo systemctl disable nginx
EOF
'''
            }

          }
        }

        stage('stop cron') {
          steps {
            withCredentials(bindings: [sshUserPrivateKey(credentialsId: 'cb4692c9-04ae-47c8-b0de-869adadb9466', keyFileVariable: 'sshkey')]) {
              sh '''ssh -o "StrictHostKeyChecking=no" -i $sshkey root@192.168.86.100 <<\'EOF\'
# crontab -r
EOF
'''
            }

          }
        }

      }
    }

  }
  environment {
    Test = ''
  }
}
