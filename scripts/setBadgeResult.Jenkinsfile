properties([
  parameters([
    string(name: 'repo', description: 'The repository the badge result is for'),
    string(name: 'branch', defaultValue: 'main', description: 'The branch in the respository the badge result is for'),
    string(name: 'label', description: 'The name the badge should have'),
    string(name: 'message', description: 'The value the badge should have'),
    choice(name: 'color', choices: ['brightgreen', 'green', 'yellowgreen', 'yellow', 'orange', 'red', 'blue', 'lightgrey'], description: 'The color the badge should have')
  ])
])

node {
  def packageJson
  def workDir = "${WORKSPACE}/${env.BRANCH_NAME}-${env.BUILD_ID}"
  def nodeImage = 'node:16'
  def version
  def exceptionThrown = false
  try {
    ansiColor('xterm') {
      dir(workDir) {

        stage('Pull Runtime Image') {
          sh "docker pull ${nodeImage}"
        }

        docker.image(nodeImage).inside() {

          parallel (
            'env': {
              stage('Runtime Versions') {
                sh 'node --version'
                sh 'npm --version'
              }
            },
            'proj': {
              stage('Checkout') {
                checkout scm
              }

              stage('Install') {
                sh 'npm ci'
              }
            }
          )

          stage('Set Build Number') {
            packageJson = readJSON file: 'package.json'
            version = "${packageJson.version}+${env.BUILD_ID}"
            currentBuild.displayName = version
          }

          stage('Run') {
            sh """npm start -- \
              --repo='${params.repo}' \
              --branch='${params.branch}' \
              --label='${params.label}' \
              --message='${params.message}' \
              --color='${params.color}' \
            """
          }

          stage('Validate Results') {
            sh 'npm run validate:results'
          }

          stage('Commit') {
            sh 'git config --global user.email "adamboe@outlook.com"'
            sh 'git config --global user.name "Adam Boe"'
            sh 'git status'
            sh 'git add badge-results/*'
            sh 'git status'
            sh 'git diff --cached'
            sh "git commit -m 'Set ${label} badge for ${repo}'"
            sh 'git status'
          }

          stage('Push') {
            sshagent (credentials: ['github-ssh']) {
              // sh 'git push origin main'
            }
          }
        }
      }
    }
  } catch (err) {
    exceptionThrown = true
    println 'Exception was caught in try block of jenkins job.'
    println err
  } finally {
    stage('Cleanup') {
      try {
        sh "rm -rf ${workDir}"
      } catch (err) {
        println 'Exception deleting working directory'
        println err
      }
      try {
        sh "rm -rf ${workDir}@tmp"
      } catch (err) {
        println 'Exception deleting temporary working directory'
        println err
      }
      if (exceptionThrown) {
        error('Exception was thrown earlier')
      }
    }
  }
}