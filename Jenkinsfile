pipeline {
  agent {
    node {
      label 'community_tester'
    }
  }
  stages {
    stage('prepareRepos') {
      agent {
        node {
          label 'community_tester'
        }
      }
      steps {
        sh '${WORKSPACE}/scripts/prepare_repos.sh'
      }
    }
    stage('checkLicenses') {
      agent {
        node {
          label 'community_tester'
        }
      }
      steps {
        sh '${WORKSPACE}/scripts/check_licenses.sh'
      }
    }
    stage('build') {
      parallel {
        stage('BuildUbuntu') {
          agent {
            node {
              label 'community_tester'
            }
          }
          steps {
            sh '${WORKSPACE}/scripts/run_test.sh ubuntu'
            sh 'mkdir out-$BUILD_NUMBER'
            sh 'cd karamel-chef && vagrant scp ubuntu-$BUILD_NUMBER:/home/vagrant/test_report/* ../out-$BUILD_NUMBER'
            sh 'cd karamel-chef && vagrant scp ubuntu-$BUILD_NUMBER:/home/vagrant/test_report/ut/* ../out-$BUILD_NUMBER/ut'
          }
          post {
            always {
              stash(name: "ubuntu-${env.BUILD_NUMBER}", includes: "out-${env.BUILD_NUMBER}/*.xml")
              junit "out-${env.BUILD_NUMBER}/ut/*.xml"
              sh 'rm -rf out-$BUILD_NUMBER'
              sh '${WORKSPACE}/scripts/shutdown.sh ubuntu-$BUILD_NUMBER'
            }
          }
        }
        stage('BuildCentos') {
          agent {
            node {
              label 'community_tester'
            }
          }
          steps {
            sh '${WORKSPACE}/scripts/run_test.sh centos'
            sh 'mkdir out-$BUILD_NUMBER'
            sh 'cd karamel-chef && vagrant scp centos-$BUILD_NUMBER.0:/home/vagrant/test_report/* ../out-$BUILD_NUMBER'
          }
          post {
            always {
              stash(name: "centos-${env.BUILD_NUMBER}", includes: "out-${env.BUILD_NUMBER}/*.xml")
              sh 'rm -fr out-$BUILD_NUMBER'
              sh '${WORKSPACE}/scripts/shutdown.sh centos-$BUILD_NUMBER.0'
              sh '${WORKSPACE}/scripts/shutdown.sh centos-$BUILD_NUMBER.1'
              sh '${WORKSPACE}/scripts/shutdown.sh centos-$BUILD_NUMBER.2'
            }
          }
        }
      }
    }
    stage('cleanup') {
      agent {
        node {
          label 'community_tester'
        }
      }
      steps {
        sh '${WORKSPACE}/scripts/clean_branches.sh'
      }
    }
  }
  post {
    always {
      unstash "ubuntu-${env.BUILD_NUMBER}"
      unstash "centos-${env.BUILD_NUMBER}"
      junit "out-${env.BUILD_NUMBER}/*.xml,out-${env.BUILD_NUMBER}/centos.xml"
      sh 'rm -r out-$BUILD_NUMBER'
    }
  }
}
