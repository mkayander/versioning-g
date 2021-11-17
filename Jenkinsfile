pipeline {
    agent any
    environment {
        BB_TOKEN  = credentials('Versioning_Tests_SemRel')
    }

    stages {
        // grouping of steps based on a specific agent
        stage("Pool Linux_Default") {

            agent {
              docker {
                label 'docker'
                image 'node:14.17.6-alpine3.13'
              }
            }

            stages {
              stage('Build and test current branch') {
                steps {
                    script {
                      if (fileExists('package-lock.json')) {
                        sh 'npm ci'
                      } else {
                        sh 'npm install'
                      }
                      sh 'npm build'
                      sh 'npm test'
                    }
                }
              }

              stage('Release') {
                // Run optional required steps before releasing
                steps {
                  // echo 'hello'
                  sh 'git --version'
                  sh 'npx semantic-release'
                }
              }
            }

            post {
                cleanup {
                    // clean up our workspace
                    cleanWs()
                }
            }
        }
    }

    post {
        always {
            echo "currentBuild.result = ${currentBuild.result}"

            script {
                if (env.BRANCH_NAME.startsWith("PR-")) {
                    def body = '''
$BRANCH_NAME from "$CHANGE_BRANCH" to "$CHANGE_TARGET" completed. Status: ''' + "${currentBuild.result}" + '''\n
build url: $BUILD_URL
sonar report: ${BUILD_LOG_REGEX, regex=".*ANALYSIS SUCCESSFUL, you can browse (.*)", showTruncatedLines=false, substText="$1"}
\n
'''
                }
            }
        }

        success {
            echo 'I succeeded!'
        }
        failure {
            echo 'Build failure! Collecting error statistics...'
        }
    }

}
