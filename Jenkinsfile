@Library("centreon-shared-library") _

pipeline {
  agent { label 'aws' } 
  stages {
    stage('AS400 BUILD AND PACKAGING') {
      parallel {
        stage('Building AS400') {
          agent { label 'aws' }
          steps {
            echo "BUILD AS400"
            sh 'docker run -i --entrypoint /src/ci/as400-build.sh -v "$PWD:/src" registry.centreon.com/as400:centos7'
          }
        }
        stage('Testing AS400') {
          agent { label 'aws' }
          steps {
            echo "BUILD AS400"
            sh 'docker run -i --entrypoint /src/ci/as400-test.sh -v "$PWD:/src" registry.centreon.com/as400:centos7'
          }
        }
        stage('Packaging AS400 for centos7') {
          environment {
            BUILD_NUMBER = "${env.BUILD_NUMBER}"
          }
          agent { label 'aws' }
          steps {
            echo "Packaging AS400 CENTOS7"
            sh '''docker run -i --entrypoint /src/ci/as400-packaging.sh -v "$PWD:/src" -e RELEASE=$BUILD_NUMBER registry.centreon.com/as400:centos7'''  
            sh 'sudo rpmsign --addsign noarch/*.rpm'
            stash name: 'el7-rpms', includes: 'noarch/*.rpm'
            archiveArtifacts artifacts: "noarch/*.rpm"
            sh 'sudo rm -rf noarch'
          }
        }
        stage('Packaging AS400 for centos8') {
          environment {
            BUILD_NUMBER = "${env.BUILD_NUMBER}"
          }
          agent { label 'aws' }
          steps {
            echo "Packaging AS400 CENTOS8"
            sh '''docker run -i --entrypoint /src/ci/as400-packaging.sh -v $PWD:/src -e RELEASE=$BUILD_NUMBER registry.centreon.com/as400:centos8'''     
            sh 'sudo rpmsign --addsign noarch/*.rpm'
            stash name: 'el8-rpms', includes: 'noarch/*.rpm'
            archiveArtifacts artifacts: "noarch/*.rpm"
            sh 'sudo rm -rf noarch'
          }
        }
      }
    }
    stage('RPM Delivery to testing repos') {
      environment {
        BUILD_NUMBER = "${env.BUILD_NUMBER}"
      }
      agent { label 'aws' }
      steps {
        echo "Deliver RPMs AS400"
        unstash 'el7-rpms'
        unstash 'el8-rpms'
        loadCommonScripts()
        sh 'ci/as400-delivery.sh testing'
      }
    }
    stage('RPM Delivery') {
      when { branch 'master' }       
      environment {
        BUILD_NUMBER = "${env.BUILD_NUMBER}"
      }
      agent { label 'aws' }
      steps {
        echo "Deliver RPMs AS400"
        unstash 'el7-rpms'
        unstash 'el8-rpms'
        loadCommonScripts()
        sh 'ci/as400-delivery.sh stable'
      }
    }
  }
  post { 
    always { 
      cleanWs()
    }
  }
}
