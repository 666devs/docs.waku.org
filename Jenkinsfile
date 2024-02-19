#!/usr/bin/env groovy
library 'status-jenkins-lib@v1.8.8'

pipeline {
  agent { label 'linux' }

  options {
    disableConcurrentBuilds()
    /* manage how many builds we keep */
    buildDiscarder(logRotator(
      numToKeepStr: '20',
      daysToKeepStr: '30',
    ))
  }

  environment {
    GIT_COMMITTER_NAME = 'status-im-auto'
    GIT_COMMITTER_EMAIL = 'auto@status.im'
    PROD_SITE = 'docs.waku.org'
    DEV_SITE  = 'dev-docs.waku.org'
    DEV_HOST  = 'jenkins@node-01.do-ams3.sites.misc.statusim.net'
    SCP_OPTS  = 'StrictHostKeyChecking=no'
  }

  stages {
    stage('Install') {
      steps {
        sh "yarn install"
      }
    }

    stage('Build') {
      steps { script {
        sh 'yarn build'
        dir('build') {
          sh "echo ${env.PROD_SITE} > CNAME"
          jenkins.genBuildMetaJSON()
        } }
      }
    }

    stage('Publish Prod') {
      when { expression { env.GIT_BRANCH ==~ /.*master/ } }
      steps {
        sshagent(credentials: ['status-im-auto-ssh']) {
          sh "ghp-import -p build"
        }
      }
    }

    stage('Publish Devel') {
      when { expression { env.GIT_BRANCH ==~ /.*develop/ } }
      steps {
        sshagent(credentials: ['jenkins-ssh']) {
          sh """
            rsync -e 'ssh -o ${SCP_OPTS}' -r --delete build/. \
              ${env.DEV_HOST}:/var/www/${env.DEV_SITE}/
          """
        }
      }
    }
  }

  post {
    cleanup { cleanWs() }
  }
}
