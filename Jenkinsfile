pipeline {
  agent { label "linux" }
  options { disableConcurrentBuilds() }

  stages {
    stage("build") {
      steps {
        sh "${WORKSPACE}/mvnw --settings settings.xml -B clean deploy -DskipTests=true"
      }
    }
  } // stages
} // end pipeline
