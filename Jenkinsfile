def getGitBranchName() {
  if (env.CHANGE_BRANCH) {
    return env.CHANGE_BRANCH
  }
  return scm.branches[0].name
}

node("uptycs") {
  stage("checkout") {
    PULL_REQUEST = env.CHANGE_ID
    GIT_BRANCH = getGitBranchName()
    if ("${GIT_BRANCH}" == "mulesoft") {
      GIT_BRANCH = "production"
    }
    cleanWs()
    git branch: GIT_BRANCH, credentialsId: 'github', url: 'https://github.com/Uptycs/uptycs-hive.git'
    GIT_COMMIT = sh (script: "git log -n 1 --pretty=format:'%H'", returnStdout: true)
  } // checkout

  withEnv(["UPTYCS_HIVE_HOME=${WORKSPACE}"]) {
    stage("info") {
      DATE = new Date().format("yyyy-MM-dd'T'HH:mm:ss'Z'")
      if ("${GIT_BRANCH}" == "release" || "${GIT_BRANCH}" == "production") {
        env.VERSION = "${BUILD_NUMBER}"
      } else {
        env.VERSION = "1.0-" + "${GIT_BRANCH}".replace("origin/", "").replace("/", "-")
      }

      println "Build version: ${VERSION}"
      println "Build number: ${BUILD_NUMBER}"
      println "Commit: ${GIT_COMMIT}"
      println "Branch: ${GIT_BRANCH}"
      println "Date/time: ${DATE}"

      sh "set"
    } // info stage

    stage("build") {
      dir("${WORKSPACE}") {
        sh "mvn clean package -Pdist -DskipTests -Dmaven.javadoc.skip=true -Drat.skip=true"
      } // dir
    } // build stage


    stage("upload-to-s3") {
      BUCKET_NAME = "uptycs-builds-2"
      if (PULL_REQUEST != null) {
        return
      }
      dir("${WORKSPACE}/packaging/target") {
        withAWS(credentials:'S3CREDS') {
          sh """
            ls -l *.tar.gz
            aws configure set default.s3.signature_version s3v4
            aws s3 cp apache-hive-2.3.7u1-bin.tar.gz s3://${BUCKET_NAME}/ --quiet
          """
        } // withAWS
      } // dir
    } // upload-to-s3
  } // withEnv
} // node
