describeNode = "echo \"Running on \${NODE_NAME} (executor: \${EXECUTOR_NUMBER})\""

pipeline {
  agent any
  environment {
    DOCKER_REGISTRY = ""
    GOOGLE_APPLICATION_CREDENTIALS = "/home/ubuntu/gcp-creds.json"
    IMAGE_TYPE = sh(script: "printf ${env.BRANCH_NAME} | sed -r 's/\\//_/g' | sed -r 's/\\./-/g'", returnStdout: true)
  }
  stages {
    stage('Setup') {
      steps {
        sh "${describeNode}"
        sh script: '''
virtualenv --python="$(command -v python3.6)" --no-site-packages venv
cat /home/ubuntu/docker-hub-password.txt | docker login -u determinedaidev --password-stdin
'''
      }
    }
    stage('Build') {
      steps {
        sh "${describeNode}"
        sh script: '''
. venv/bin/activate
make clean all
'''
      }
    }
    stage('Push') {
      steps {
        sh "${describeNode}"
        sh script: '''
. venv/bin/activate
make publish-dev
'''
      }
    }
    stage('Deploy') {
      steps {
        sh script: """
. venv/bin/activate
det-deploy aws --user ${env.BRANCH_NAME} --version `git rev-parse HEAD` --keypair integrations-test
"""
      }
    }
  }
}