def config = [:]
def env = [:]
def pom

pipeline {
  agent any
  triggers {
    pollSCM('H/2 * * * *')
  }

  parameters {
    booleanParam(name: 'DEPLOY_TO_EXCHANGE', defaultValue: true, description: 'Deploy to Anypoint Exchange?')
  }

  environment {
    ANYPOINT_CREDENTIALS = credentials('efd47c2c-1afe-44f3-819e-4f246697cfb5')
  }

  stages {

    stage("Configuration setup...") {
      steps {
        echo "Loading environment config..."
        script {
          config = readJSON file: "env/${env.BRANCH_NAME}/config.json"
          env = config.envConfig
          pom = readMavenPom file: 'pom.xml'
        }
      }
    }

    stage('Build Application') {
      steps {
        echo "Building application..."
        sh 'mvn clean install -s build-config/settings.xml'
      }  
    }

    stage('Deploy to Exchange') {
      when {
        expression { return params.DEPLOY_TO_EXCHANGE }
      }
      steps {
        script {
          def artifactExists = sh(
            script: """curl -s -o /dev/null -w "%{http_code}" -u ${ANYPOINT_CREDENTIALS_USR}:${ANYPOINT_CREDENTIALS_PSW} \
            https://anypoint.mulesoft.com/exchange/api/v2/assets/${pom.groupId}/${pom.artifactId}/${pom.version}""",
            returnStdout: true
          ).trim()

          if (artifactExists == "200") {
            echo "Artifact ${pom.artifactId} version ${pom.version} already exists in Exchange. Skipping deploy."
          } else {
            echo "Deploying artifact to Exchange..."
            sh 'mvn deploy -s build-config/settings.xml'
          }
        }
      }  
    }

    stage('Deploy to CloudHub 2.0') {
      steps {
        echo "Deploying to CloudHub 2.0..."
        sh """
          mvn deploy -s build-config/settings.xml \
            -DmuleDeploy \
            -Dcloudhub2Deployment \
            -DdeploymentTarget=${env.deploymentTargetId} \
            -Dreplicas=${env.replicas} \
            -DvCores=${env.vCores} \
            -DworkerType=${env.workerType} \
            -Dmule.version=${env.muleVersion} \
            -DcloudhubAppName=${env.cloudhubAppName} \
            -Dcloud.user=${ANYPOINT_CREDENTIALS_USR} \
            -Dcloud.password=${ANYPOINT_CREDENTIALS_PSW}
        """
      }
    }

  }

  post {
    failure {
      echo "Pipeline failed. Please check the logs above."
    }
    success {
      echo "Pipeline completed successfully."
    }
  }
}
