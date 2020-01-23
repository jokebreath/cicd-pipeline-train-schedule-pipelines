pipeline {
  agent any
  stages {
    stage('build') {
      steps {
        echo 'Running build automation'
          sh './gradlew build --no-daemon'
          archiveArtifacts artifacts: 'dist/trainSchedule.zip'
      }
    }
    stage('DeployToStaging') {
      when {
        branch 'master'
      }
      steps {
        withCredentials([usernamePassword(credentialsId: 'deploy_user', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
          sshPublisher(
            failOnError: true,
            continueOnError: false,
            publishers: [
              sshPublisherDesc(
                configName: 'staging',
                sshCredentials: [
                  username: "$USERNAME",
                  encryptedPassphrase: "$PASSWORD"
                ],
                transfers: [
                  sshTransfer(
                    sourceFiles: 'dist/trainSchedule.zip',
                    removePrefix: 'dist/',
                    remoteDirectory: '/tmp',
                    execCommand: 'sudo /usr/bin/systemctl stop train-schedule && rm -rf /opt/train-schedule/* && unzip /tmp/trainSchedule.zip -d /opt/train-schedule && sudo /usr/bin/systemctl start train-schedule'
                  )
                ]
              )
            ]
          )
        }
      }
      stage('deployToProduction') {
        when {
          branch 'master'
        }
        steps {
          input 'Are you sure?'
          milestone(1)
          withCredentials([usernamePassword(credentialsId: 'deploy_user', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
            sshPublisher(
              failOnError: true,
              continueOnError: false,
              publishers: [
                sshPublishderDesc(
                  configName: 'production',
                  sshCredentials: [
                    username: "$USERNAME",
                    encryptedPassphrase: "$PASSWORD"
                  ],
                  transfers: [
                    sshTransfer(
                      sourceFiles: 'dist/trainSchedule.zip',
                      removePrefix: 'dist/',
                      remoteDirectory: '/tmp',
                      execCommand: 'sudo /usr/bin/systemctl stop train-schedule && rm -rf /opt/train-schedule/* && unzip /tmp/trainSchedule.zip -d /opt/train-schedule && sudo /usr/bin/systemctl start train-schedule'
                    )
                  ]
                )
              ]
            )
          }
        }
      }
    }
  }
}
