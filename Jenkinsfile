pipeline {
  agent any
  
  triggers {
    // Remove polling as we'll use webhooks
    // pollSCM('* * * * *')
  }
  
  stages {
    stage('Checkout & Prepare') {
      steps {
        sh 'echo Hello World'
        checkout([
          $class: 'GitSCM',
          branches: [[name: env.CHANGE_BRANCH ?: env.BRANCH_NAME]],  // Use PR branch if available
          extensions: [
            [$class: 'PreBuildMerge',
             options: [
               fastForwardMode: 'FF',
               mergeTarget: env.CHANGE_TARGET ?: env.BRANCH_NAME,
               mergeStrategy: 'default',
               remote: env.CHANGE_FORK ?: env.GIT_URL,
               refspec: "+refs/heads/${env.CHANGE_BRANCH}:refs/remotes/origin/${env.CHANGE_BRANCH}"
             ]
            ]
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
          // Update GitHub PR status
          updateGitHubCommitStatus(
            state: 'SUCCESS',
            context: 'jenkins/unit-test',
            description: 'Unit tests passed successfully'
          )
          
          // If this is a PR, add a comment
          if (env.CHANGE_ID) {
            def comment = """
              ✅ Unit tests passed successfully!
              This PR is ready to be merged.
            """.stripIndent()
            
            githubNotify(
              issueNumber: env.CHANGE_ID,
              comment: comment
            )
          }
        }
        failure {
          // Update GitHub PR status
          updateGitHubCommitStatus(
            state: 'FAILURE',
            context: 'jenkins/unit-test',
            description: 'Unit tests failed'
          )
          
          // If this is a PR, add a comment
          if (env.CHANGE_ID) {
            def comment = """
              ❌ Unit tests failed!
              Please fix the failing tests before merging.
            """.stripIndent()
            
            githubNotify(
              issueNumber: env.CHANGE_ID,
              comment: comment
            )
          }
        }
      }
    }
  }
  
  post {
    always {
      // Clean workspace after build
      cleanWs()
    }
  }
}