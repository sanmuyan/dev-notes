# Pipline

## Jenkinsfile 模板

```groovy
pipeline {
  agent {
    node {
      label 'base'
    }
  }
  environment {
    // 全局变量
    FILE_URL = 'https://www.baidu.com'
    ROOT_PASSWORD = credentials('root-password')
    KUBECONFIG = """${sh(returnStdout: true, script: 'echo "~/.kube/config"').trim()}"""
  }
  options {
    // 全局超时时间
    timeout(time: 5, unit: 'MINUTES')
  }
  parameters {
    string(name: 'APP_NAME', defaultValue: 'api-gateway', description: '应用名称')
    choice(name: 'ENV_NAME', choices: ['dev', 'prod'], description: '环境名')
  }
  stages {
    stage('Pre stage') {
      when {
        environment name: 'ENV_NAME', value: 'prod'
      }
      steps{
        input(message: '需要审批', ok: '批准', submitter: 'admin, zhangsan')  
      }
    }
    stage('Init stage') {
      steps {
        git(credentialsId: 'root-ssh', url: 'git@github.com:sanmuyan/dev-notes.git', branch: 'master', changelog: true, poll: false)
        script {
            env.IMG = sh(returnStdout: true, script: 'echo "test"').trim()
        }
      }
    }
    stage('Build stage') {
     options {
        // 阶段超时时间
        timeout(time: 1, unit: 'MINUTES')
      }
      steps {
        // 步骤重试次数
        retry(3) {
          sh 'pwd'
        }
        //步骤超时
        timeout(time: 3, unit: 'SECONDS') {
          sh 'sleep 1'
        }   
      }
    }
    stage('Test stage') {
      // 并行步骤
      parallel {
        stage('Test 1') {
          steps {
            echo "test 1"
            sh 'echo "root 密码 $ROOT_PASSWORD"'
            script {
              if (env.ENV_NAME == 'dev') {
                echo "开发环境"
              }else {
                echo "生产环境"
              }
            }
          }
        }
        stage('Test 2') {
          when {
            // 否定条件
            not {
              environment name: 'ENV_NAME', value: 'dev'
            }
            // 全部条件满足
            allOf {
              environment name: 'ENV_NAME', value: 'dev'
            }
            // 一个条件满足
            anyOf {
              branch 'dev'
            }
          }
          steps {
            echo "test 2"
          }
        }
      }
    }
  }
  post {
    always {
      echo '总是执行'
    }
    success {
      echo '成功时执行'
    }
    failure {
      echo '失败时执行'
    }
    unstable {
      echo '不稳定时执行'
    }
    changed {
      echo '变更时执行'
    }
    aborted {
      echo '取消时执行'
    }
  }
}
```
