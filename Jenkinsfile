pipeline {
  agent any
  stages {
    stage('Checkout & Prepare') {
      steps {
        sh 'echo Hello World'
        checkout([
          $class: 'GitSCM',
          branches: [[name: env.GIT_BRANCH]],  // 检出 PR 分支
          extensions: [
            // 处理 PR 源分支（尤其是 fork 仓库）
            [$class: 'PreBuildMerge', options: [mergeTarget: env.CHANGE_TARGET]]
          ],
          userRemoteConfigs: [[url: env.GIT_URL]]
        ])
      }
    }
    stage('Unit Test') {
      steps {
        sh 'mvn clean test'  // 根据项目替换为实际测试命令（如 mvn test、pytest 等）
      }
      post {
        success {
          // 测试通过时，通知 GitHub 状态为 success
          updateGitHubCommitStatus(
            state: 'SUCCESS',
            context: 'jenkins/unit-test'
          )
        }
        failure {
          // 测试失败时，通知 GitHub 状态为 failure
          updateGitHubCommitStatus(
            state: 'FAILURE',
            context: 'jenkins/unit-test'
          )
        }
      }
    }
  }
}