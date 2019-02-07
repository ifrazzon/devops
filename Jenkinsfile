node('docker') {
  deleteDir()

  stage 'Checkout'
  checkout scm

  def buildEnv = docker.image('ruby:latest')
  buildEnv.pull()
  buildEnv.inside {
    stage 'D/L dependencies'
    sh 'bundle'

    stage 'Build'
    sh 'rake build:production'

    stage 'Test'
    try {
      sh 'rake test:production'
    } catch (err) {
      currentBuild.result = 'UNSTABLE'
    }

    stage 'Build production'
    sh 'rake build:production'
    archive 'output/**'

    stash includes: 'output/**', name: 'built-site'
  }
}

if (currentBuild.result == 'UNSTABLE') {
  echo 'Skipping deployment due to unstable build'
} else {

  stage 'Deploy'
  node() {
    deleteDir()
    unstash 'built-site'

    withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'website', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME']]) {
      sh 'SSHPASS=$PASSWORD sshpass -e ssh -l $USERNAME ${env.HOST} rm -rf /var/www/vhosts/sketchingdev.co.uk/httpdocs/*'
      sh 'SSHPASS=$PASSWORD sshpass -e scp -r output/* $USERNAME@${env.HOST}:/var/www/vhosts/sketchingdev.co.uk/httpdocs'
    }
  }
}
