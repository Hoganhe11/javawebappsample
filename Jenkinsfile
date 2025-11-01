import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv([
    'AZURE_SUBSCRIPTION_ID=3a08d5f2-f291-4e2f-a8ac-5db510eeda18',
    'AZURE_TENANT_ID=b7df592e-345a-402d-8ed0-08cdd72ceb56'
  ]) {

    stage('init') {
      checkout scm
    }

    stage('build') {
      // this is what the sample project uses
      sh 'mvn clean package'
    }

    stage('deploy') {
      def resourceGroup = 'jenkins-get-started-rg'
      def webAppName   = 'jenkins-hexiaoxu-app'
      withCredentials([
        usernamePassword(
          credentialsId: 'jenkins-azure-sp',
          passwordVariable: 'AZURE_CLIENT_SECRET',
          usernameVariable: 'AZURE_CLIENT_ID'
        )
      ]) {
        sh """
          az login --service-principal \
            -u $AZURE_CLIENT_ID \
            -p $AZURE_CLIENT_SECRET \
            -t $AZURE_TENANT_ID

          az account set --subscription $AZURE_SUBSCRIPTION_ID
        """
      }

      def pubProfilesJson = sh(
        script: "az webapp deployment list-publishing-profiles -g ${resourceGroup} -n ${webAppName} --output json",
        returnStdout: true
      ).trim()

      def ftpProfile = getFtpPublishProfile(pubProfilesJson)
      sh "curl -T target/calculator-1.0.war ${ftpProfile.url}/webapps/ROOT.war -u '${ftpProfile.username}:${ftpProfile.password}'"
      sh 'az logout'
    }
  }
}

// ===== helper =====
def getFtpPublishProfile(String jsonText) {
  def json = new groovy.json.JsonSlurper().parseText(jsonText)
  // in Web App publish profiles, FTP profile usually has publishMethod == "FTP"
  def ftp = json.find { it.publishMethod == 'FTP' }
  return [
    url     : "ftp://${ftp.publishUrl}",
    username: ftp.userName,
    password: ftp.userPWD
  ]
}
