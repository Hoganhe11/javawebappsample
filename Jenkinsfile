import groovy.json.JsonSlurper

// ---- helper: keep ONLY ONE of these ----
def getFtpPublishProfile(String jsonText) {
  def profiles = new JsonSlurper().parseText(jsonText)
  def ftp = profiles.find { it.publishMethod == 'FTP' }
  return [
    // Azure gives publishUrl without protocol, so add ftp://
    url     : "ftp://${ftp.publishUrl}",
    username: ftp.userName,
    password: ftp.userPWD
  ]
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
      // your Jenkins node doesn't have mvn, so run Maven in a container
      // (this is the same idea as Jenkinsfile_container_plugin)
      docker.image('maven:3.9.9-eclipse-temurin-17').inside {
        sh 'mvn -B -DskipTests clean package'
      }
    }

    stage('deploy') {
      def resourceGroup ='jenkins-get-started-rg'
      def webAppName   = 'jenkins-hexiaoxu-app'

      // use the service principal you created in Jenkins
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

      // get publish profiles from the web app
      def pubProfilesJson = sh(
        script: "az webapp deployment list-publishing-profiles -g ${resourceGroup} -n ${webAppName} --output json",
        returnStdout: true
      ).trim()

      def ftpProfile = getFtpPublishProfile(pubProfilesJson)

      // upload the built WAR (make sure this WAR name matches your pom!)
      sh "curl -T target/calculator-1.0.war ${ftpProfile.url}/webapps/ROOT.war -u '${ftpProfile.username}:${ftpProfile.password}'"

      sh 'az logout'
    }
  }
}
