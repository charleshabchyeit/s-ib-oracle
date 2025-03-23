def config = {}
def env = {}
pipeline {
  agent any
  triggers{
    pollSCM('H/2 * * * *')
  }
  stages {
    stage("Configuration setup..."){
            steps{
                echo "Configuration setup..."
                script{
                    config = readJSON file:"env/${env.BRANCH_NAME}/config.json"
                    env = config.get("envConfig")
                }
            }
    }
    
    stage('Build Application') {
      steps {
        configFileProvider([configFile(fileId: 'custom-settings', variable: 'MAVEN_SETTINGS')]) {
          sh 'mvn clean install -s $MAVEN_SETTINGS'
        } 
      }
    }

    stage('Deploy CloudHub') {
      environment {
        ANYPOINT_CREDENTIALS = credentials('efd47c2c-1afe-44f3-819e-4f246697cfb5')
      }
      steps {
        sh "mvn deploy -DmuleDeploy -Dcloud.env=${env.envName} -DcloudhubAppName=${env.cloudhubAppName} -Dmule.version=${env.muleVersion} -Dcloud.user=${ANYPOINT_CREDENTIALS_USR} -Dcloud.password=${ANYPOINT_CREDENTIALS_PSW}"
      }
    }
  }
}

