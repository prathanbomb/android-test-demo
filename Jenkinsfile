pipeline {
  agent any
  stages {
    stage('build') {
      steps {
        sh './gradlew clean assemble assembleAndroidTest'
      }
    }
    stage('unit test') {
      steps {
        sh './gradlew test'
      }
    }
    stage('packaging') {
      steps {
        archiveArtifacts(artifacts: '**/*.apk', fingerprint: true)
        junit(testResults: '**/*.xml', allowEmptyResults: true)
      }
    }
    stage('ui test') {
      steps {
        sh 'gcloud firebase test android run firebase-test-matrix.yml:instrumentation-test --app app/build/outputs/apk/mock/debug/app-mock-debug.apk --test app/build/outputs/apk/androidTest/mock/debug/app-mock-debug-androidTest.apk --project digioci'
        sh 'gcloud firebase test android run firebase-test-matrix.yml:robo-test --app app/build/outputs/apk/prod/release/app-prod-release.apk --project digioci'
      }
    }
    stage('publish') {
      steps {
        androidApkUpload apkFilesPattern: '**/app-prod-release.apk', googleCredentialsId: 'Google Play Android Developer', rolloutPercentage: '100', trackName: 'internal'
      }
    }
  }
  post {
    failure {
      sh "curl https://notify-api.line.me/api/notify -H 'Authorization: Bearer 9VtlAqMLsbQbozGBvanO96aPHrLU901TJKpW8ErlTHU' -F 'message=${env.JOB_NAME} #${env.BUILD_NUMBER} \nresult is failed. \n${env.BUILD_URL}' -F 'stickerId=173' -F 'stickerPackageId=2'"
    }
    success {
      sh "curl https://notify-api.line.me/api/notify -H 'Authorization: Bearer 9VtlAqMLsbQbozGBvanO96aPHrLU901TJKpW8ErlTHU' -F 'message=${env.JOB_NAME} #${env.BUILD_NUMBER} \nresult is success. \n${env.BUILD_URL}' -F 'stickerId=525' -F 'stickerPackageId=2'"
    }
  }
}
