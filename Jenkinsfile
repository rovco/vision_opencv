pipeline {
  agent any

  options {
    skipDefaultCheckout true
  }

  environment {
    JENKINS_CREDS = credentials('Jenkins-API-Key')
    NOTE=""
  }

  stages {

    stage( checkout ) {
      steps {
        dir('source') {
            checkout scm
        }
      }
    }

    stage( build ) {
      steps {
        sh "ciDocker build"
      }
    }

    stage( test ) {
      steps {
        ansiColor('xterm') {
          sh "ciDocker test"
        }
      }
      post {
        always {
          junit allowEmptyResults: true, testResults: 'build/catkin_ws/build/**/*.xml'
          script {
            try {
              benchmark altInputSchema: '', altInputSchemaLocation: "${env.JENKINS_HOME}/rvco_citools/links/benchmark.schema.json", inputLocation: 'build/catkin_ws/*.json', schemaSelection: 'customSchema', truncateStrings: true
            } catch (err) {
              echo "There are no benchmark"
            }
          }

          ansiColor('xterm') {
            sh "ciDocker post_test API_USER=$JENKINS_CREDS_USR API_TOKEN=$JENKINS_CREDS_PSW API_URL=$JOB_URL"
          }
          archiveArtifacts artifacts: 'build/metric_plots/', onlyIfSuccessful: true, allowEmptyArchive: true
          archiveArtifacts artifacts: 'build/catkin_ws/*.json', onlyIfSuccessful: true, allowEmptyArchive: true
          archiveArtifacts artifacts: 'build/catkin_ws/*.tum', onlyIfSuccessful: true, allowEmptyArchive: true
        }
      }
    }
  }


  post {
    // Notifications only.
    success {
      script {
        NOTE = sh ( script: "fingerPointer.py --repo source", returnStdout:true ).trim()
      }
      slackSend color: 'good', message: "Successfully built ${env.BUILD_TAG}. Yay!\n${NOTE}"
    }
    failure {
      script {
        NOTE = sh ( script: "fingerPointer.py --repo source --poke", returnStdout:true ).trim()
      }
      slackSend color: 'bad', message: "Failure building ${env.BUILD_TAG}.\n${env.BUILD_URL}\n${NOTE}"
    }
    
    //Clean workspace.
    cleanup {
      echo 'Cleaning workspace'
      deleteDir()
    }
  }

}

