#!groovy

properties(
    [    // To discard old build
        [$class: 'BuildDiscarderProperty', strategy:
        // Manages how long to keep records of the builds.
          [$class: 'LogRotator', artifactDaysToKeepStr: '14', artifactNumToKeepStr: '5', daysToKeepStr: '30', numToKeepStr: '60']],
        pipelineTriggers(
          [
              pollSCM('H/15 * * * *'),
              cron('@daily'),
          ]
        )
    ]
)
node {

    stage('Checkout') {
        // disable to recycle workspace data to save time/bandwidth
        deleteDir()
        checkout scm
    }

    docker.image('trion/ng-cli-karma:1.2.1').inside {
      stage('NPM Install') {
          // silent warn messages
          withEnv(["NPM_CONFIG_LOGLEVEL=warn"]) {
              sh 'npm install'
          }
      }

      stage('Test') {
          withEnv(["CHROME_BIN=/usr/bin/chromium-browser"]) {
            sh 'ng test --progress=false --watch false'
          }
          // add a reporter that creates JUnit XML reports
          junit '**/test-results.xml'
      }

      stage('Lint') {
          // add a reporter that creates JUnit XML reports
          sh 'ng lint'
      }
        
      stage('Build') {
          milestone()
          sh 'ng build --prod --aot --sm --progress=false'
      }
    }
    //end docker

    stage('Archive') {
        // archive the build artifact
        sh 'tar -cvzf dist.tar.gz --strip-components=1 dist'
        archive 'dist.tar.gz'
    }

    stage('Deploy') {
        milestone()
        echo "Deploying..."
    }
}
