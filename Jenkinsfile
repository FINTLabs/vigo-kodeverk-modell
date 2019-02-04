pipeline {
  agent none
  environment {
    GITHUB = credentials('github_fint_jenkins')
  }
  stages {
    stage('Generate Model') {
      agent { 
        docker {
          label 'docker'
          image 'golang'
        }
       }
      when {
        tag pattern: "v\\d+\\.\\d+\\.\\d+(-\\w+-\\d+)?", comparator: "REGEXP"
      }
      steps {
        sh "go get github.com/FINTprosjektet/fint-model && go install github.com/FINTprosjektet/fint-model && fint-model --owner FINTlabs --repo vigo-kodeverk-modell --filename Vigo-kodeverkmodell.xml --tag ${TAG_NAME} generate"
        stash(name: 'java', includes: 'java/**')
      }
    }
    stage('Commit Java Model') {
      agent { 
        docker {
          label 'docker'
          image 'dtr.fintlabs.no/jenkins/git:latest'
        }
      }
      when {
        tag pattern: "v\\d+\\.\\d+\\.\\d+(-\\w+-\\d+)?", comparator: "REGEXP"
      }
      steps {
        script {
          VERSION = TAG_NAME[1..-1]
        }
        git 'https://github.com/FINTmodels/fint-vigokv-model.git'
        sh 'git clean -fdx'
        unstash 'java'
        sh 'rm -rf src/main/java/no/fint/model/*'
        sh 'mv java/* src/main/java/no/fint/model/'
        sh "echo version=${VERSION} > gradle.properties"
        sh 'git config user.email "jenkins@fintprosjektet.no"'
        sh 'git config user.name "FINT Jenkins"'
        sh 'git add gradle.properties src/main/java/no/fint/model/'
        sh "git commit -m 'Version ${VERSION}'"
        sh "git push 'https://${GITHUB}@github.com/FINTmodels/fint-vigokv-model.git' master:master"
      }
    }
  }
}
