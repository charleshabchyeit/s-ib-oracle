def config = [:]
def env = [:]

pipeline {
  agent any

  triggers {
    pollSCM('H/2 * * * *')
  }

  stages {

    stage("Configuration setup") {
      steps {
        echo "Loading configuration..."
        script {
          config = readJSON file: "env/${env.BRANCH_NAME}/config.json"
          env = config.get("envConfig")
        }
      }
    }

    stage("Auto-Versioning") {
      steps {
        script {
          def baseVersion = "1.0.0"
          def timestamp = new Date().format("yyyyMMdd.HHmmss")
          def newVersion = "${baseVersion}-${timestamp}"

          echo "Setting project version to: ${newVersion}"

          sh "mvn versions:set -DnewVersion=${newVersion}"
          sh "mvn versions:commit"

          writeFile file: 'version.txt', text: newVersion
        }
      }
    }

    stage("Build Application") {
      steps {
        echo "Building the MuleSoft application..."
        sh 'mvn clean install -s build-config/settings.xml'
      }
    }

    stage("Deploy to Exchange") {
      steps {
        echo "Deploying to Anypoint Exchange..."
        script {
          def version = readFile('version.txt').trim()
          sh "mvn deploy -s build-config/settings.xml -DmuleDeploy -Dproject.version=${version}"
        }
      }
    }

    stage("Deploy to CloudHub") {
      environment {
        ANYPOINT_CREDENTIALS = credentials('efd47c2c-1afe-44f3-819e-4f246697cfb5')
      }
      steps {
        echo "Deploying to CloudHub..."
        script {
          def version = readFile('version.txt').trim()
          sh """
            mvn deploy \
              -s build-config/settings.xml \
              -DmuleDeploy \
              -Dcloud.env=${env.envName} \
              -DcloudhubAppName=${env.cloudhubAppName} \
              -Dmule.version=${env.muleVersion} \
              -Dcloud.user=${ANYPOINT_CREDENTIALS_USR} \
              -Dcloud.password=${ANYPOINT_CREDENTIALS_PSW} \
              -Dproject.version=${version}
          """
        }
      }
    }
  }
}