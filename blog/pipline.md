# Pipline

```groovy
pipeline {
  agent {
    node {
      label 'base'
    }

  }
  environment {
    KUBECONFIG = """${sh(returnStdout: true, script: 'echo "${KUBECONFIG_PATH}/${CLUSTER_NAME}"').trim()}"""
    APP_NAME = """${sh(returnStdout: true, script: 'echo "${JOB_BASE_NAME}"').trim()}"""
  }  
  stages {
    stage('get-code') {
      agent none
      steps {
        container('base') {
          git(credentialsId: 'admin-k8s-ssh', url: 'git@gitlab.ops.yangege.cn:devops/ops-tools.git', branch: '${GIT_BRANCH}', changelog: true, poll: false)
          script {
            env.IMG = sh(returnStdout: true, script: 'echo "reg.ops.yangege.cn/devops/${CLUSTER_NAME}/${JOB_BASE_NAME}:${GIT_BRANCH}-$(git rev-parse --short HEAD)"').trim()
          }

        }

      }
    }

    stage('build') {
      agent none
      steps {
        container('base') {
          sh '''
docker build -t $IMG .
docker push $IMG
'''
        }

      }
    }

    stage('deploy') {
      agent none
      steps {
        container('base') {
          sh '''
kubectl --kubeconfig $KUBECONFIG -n $NS set image deployment/$APP_NAME app=$IMG
sleep 3
try_time=0
try_number=60
rollout_command="kubectl -n $NS rollout status deployment/$APP_NAME"
until $rollout_command || [ $try_sec -eq $try_number ]; do
  $rollout_command
  try_sec=$((try_sec + 1))
  sleep 10
done

kubectl --kubeconfig $KUBECONFIG -n $NS set image cronjob/$APP_NAME-task app=$IMG
'''
        }

      }
    }
  }
}
```
